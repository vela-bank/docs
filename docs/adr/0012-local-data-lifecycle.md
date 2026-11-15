# ADR-0012 — Ciclo de vida de los datos locales

- **Estado:** Aceptado
- **Fecha:** 2026-11-15
- **Decisores:** Geronimo Acevedo
- **Relacionados:** [ADR-0005](0005-local-platform-and-floci.md), [ADR-0010](0010-application-stack.md), [ADR-0011](0011-platform-and-operations.md), [Desarrollo local](../operations/local-development.md), [Testing](../operations/testing.md)

## Contexto

El [ADR-0005](0005-local-platform-and-floci.md) incluyó RDS entre los servicios que Floci emula. A la vez, el
[ADR-0010](0010-application-stack.md) fija PostgreSQL 16 y el [desarrollo local](../operations/local-development.md)
levanta PostgreSQL en Compose, con Testcontainers para los tests. Eso deja dos caminos para la misma
base: un PostgreSQL de Compose y un RDS emulado, con configuración, credenciales y ciclo de vida
distintos.

Dos fuentes de verdad para el mismo dato producen migraciones divergentes y tests que pasan contra
el motor equivocado. Además, el RDS de Floci ejecuta un contenedor por instancia: añade tiempo de
arranque, requiere el socket de Docker y convierte a Floci en un punto de fallo del que dependería
también Keycloak.

La práctica habitual no emula la base de datos: ejecuta el motor real y reserva los emuladores para
las APIs de nube que no se pueden correr localmente.

## Decisión

Un solo motor —PostgreSQL 16— con tres ciclos de vida:

| Contexto | Mecanismo | Persistencia | Quién lo crea |
| --- | --- | --- | --- |
| Local (desarrollo) | Docker Compose | Volumen; sobrevive a `dev-down`, se borra con `dev-reset` | `make dev-up` |
| Tests (local y CI) | Testcontainers | Efímera, una instancia por suite | La propia suite |
| AWS (sandbox, staging, prod) | RDS | Gestionada por AWS | Terraform |

Reglas derivadas:

- Floci no aporta base de datos. Queda para S3, SQS/SNS, Secrets Manager y ECR.
- El RDS de Floci no se usa en el MVP. El camino de aprovisionamiento se valida en AWS sandbox.
- El esquema se define solo en las migraciones de Flyway (`db/migration`). Ningún contexto crea
  tablas a mano.
- La aplicación no distingue contextos: recibe `spring.datasource.url` por configuración, con un
  perfil por entorno (`local`, `test`, `staging`, `prod`).
- Keycloak usa su propia base en Compose y no depende de Floci.
- La versión del motor es la misma en desarrollo y tests (`postgres:16`); la de RDS se fija en
  Terraform.

## Consecuencias

### Positivas

- Un único árbitro del esquema: las mismas migraciones se aplican en los tres contextos.
- Loop local rápido y sin dependencia de Floci para trabajar con datos.
- Keycloak y la base arrancan aunque Floci esté caído.
- Menos recursos y sin socket de Docker para la base de datos.
- Coherente con lo ya decidido en el ADR-0010 y en la estrategia de testing.

### Negativas

- El aprovisionamiento de RDS no se ejercita en local; se aprende en el sandbox.
- Puede aparecer deriva entre el PostgreSQL local y RDS: TLS obligatorio, límites de conexión,
  parameter groups, extensiones.
- El Terraform de RDS se prueba más tarde y en un entorno con coste.

### Mitigaciones

- Sandbox programado con `terraform apply` / `destroy` para ejercitar RDS.
- Configuración por perfil y revisión de los parámetros de conexión (TLS, pool) antes de staging.
- Tests de integración con Testcontainers en cada PR.
- Migraciones probadas en Testcontainers antes de mergear.

## Alternativas consideradas

1. **RDS de Floci como fuente de verdad local.** Se descarta: mismo motor, más tiempo de arranque,
   socket de Docker y Floci como punto único de fallo, incluido Keycloak.
2. **Mantener Compose y RDS de Floci en paralelo.** Se descarta: dos configuraciones para el mismo
   dato y riesgo de divergencia.
3. **PostgreSQL dentro de kind (StatefulSet).** Se descarta para el MVP: añade complejidad operativa
   sin mejorar la fidelidad del motor.
4. **Conectar el desarrollo a un RDS compartido en AWS.** Se descarta: coste, latencia y riesgo de
   tocar datos compartidos.

## Reglas

- La base de datos no se emula: se ejecuta.
- Ningún documento ni script debe asumir que el RDS de Floci es necesario para desarrollar.
- Toda migración se prueba en Testcontainers antes de mergear.
- Los datos locales son sintéticos y descartables; no se copian a AWS ni desde AWS.
- Un cambio de versión de PostgreSQL se hace en los tres contextos y se registra en un ADR.

## Notas

Este ADR supersede parcialmente al [ADR-0005](0005-local-platform-and-floci.md) en lo relativo a
RDS. El resto de sus decisiones sigue vigente.
