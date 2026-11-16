# Decisiones pendientes

Las decisiones marcadas como `[PROPUESTO]` son recomendaciones, no decisiones cerradas. Para
cerrar una decisión se crea o actualiza un ADR y se elimina de esta lista.

## Tabla de decisiones

No hay decisiones abiertas que bloqueen el MVP. Las decisiones fundacionales se cerraron en los
ADRs [0008](adr/0008-repo-governance-and-naming.md), [0009](adr/0009-product-currency-and-simulation.md),
[0010](adr/0010-application-stack.md), [0011](adr/0011-platform-and-operations.md) y
[0012](adr/0012-local-data-lifecycle.md).

Las decisiones que aparezcan durante la implementación se agregan acá antes de resolverlas.

### Abiertas durante la implementación

Se resuelven cuando el desarrollo las necesite, no antes. El orden de trabajo es un microservicio
a la vez: primero uno, después el otro.

| ID | Decisión | Opciones | Recomendación `[PROPUESTO]` | Se decide cuando |
| --- | --- | --- | --- | --- |
| D-21 | Red entre pods de kind y Floci, y registry local de imágenes | A: Floci en el host, `FLOCI_HOSTNAME` y GHCR como único registry. B: Floci dentro del clúster (Helm) y ECR de Floci como registry local. | A. Coherente con D-12 (GHCR en local/CI) y evita un segundo registry. | Antes de desplegar el primer servicio en kind (walking skeleton). |
| D-22 | IAM en Floci: enforcement activo o no | A: desactivado en el loop diario y activo en un perfil dedicado (`FLOCI_SERVICES_IAM_ENFORCEMENT_ENABLED=true`). B: desactivado siempre y IAM solo en el sandbox de AWS. | A. Los tests de autorización de servicio a recurso no prueban nada de AWS con IAM apagado. | Cuando el primer servicio acceda a S3, SQS o Secrets Manager. |

Contexto de ambas: [ADR-0005](adr/0005-local-platform-and-floci.md),
[ADR-0011](adr/0011-platform-and-operations.md) y [ADR-0012](adr/0012-local-data-lifecycle.md).
Mientras estén abiertas, el desarrollo local usa Floci en el host, sin enforcement de IAM y con
credenciales descartables.

## Cómo cerrar una decisión

1. Elegir una opción concreta.
2. Escribir un ADR en `docs/adr/` con contexto, decisión y consecuencias.
3. Actualizar los documentos afectados.
4. Eliminar o marcar como cerrada la fila de esta tabla.
5. Informar en el resumen del cambio.

## Decisiones ya tomadas

| Fecha | Decisión | Evidencia |
| --- | --- | --- |
| 2026-11-09 | Trabajar con dos microservicios de negocio y multi-repo | [ADR-0001](adr/0001-polyrepo.md) |
| 2026-11-09 | Separar core banking de transfers | [ADR-0002](adr/0002-service-boundaries.md) |
| 2026-11-09 | Usar ledger de doble entrada | [ADR-0003](adr/0003-ledger.md) |
| 2026-11-09 | Contratos OpenAPI/eventos como fuente de verdad | [ADR-0004](adr/0004-contracts-and-integration.md) |
| 2026-11-09 | Floci para emulación local de AWS | [ADR-0005](adr/0005-local-platform-and-floci.md) |
| 2026-11-09 | GitOps con promoción por digest | [ADR-0006](adr/0006-gitops-and-environments.md) |
| 2026-11-09 | Protocolo de colaboración con IA | [ADR-0007](adr/0007-ai-collaboration.md) |
| 2026-11-13 | Org `vela-bank`, servicios `core-banking-service`/`transfers-service`, repos privados, licencia Apache-2.0, dominio solo local | [ADR-0008](adr/0008-repo-governance-and-naming.md) |
| 2026-11-13 | Moneda única COP (Colombia), Mailpit, KYC mock, SLO 99.9% en staging | [ADR-0009](adr/0009-product-currency-and-simulation.md) |
| 2026-11-13 | Java 21 + Spring Boot, React + TypeScript + Vite, PostgreSQL 16, SQS/SNS + outbox, Keycloak | [ADR-0010](adr/0010-application-stack.md) |
| 2026-11-13 | AWS, Terraform, kind, GHCR/ECR, Argo CD, Prometheus/Grafana/Loki/Tempo | [ADR-0011](adr/0011-platform-and-operations.md) |
| 2026-11-15 | PostgreSQL 16 en Compose/Testcontainers; RDS fuera del entorno local | [ADR-0012](adr/0012-local-data-lifecycle.md) |
| 2026-11-16 | Visibilidad pública del portfolio; `infra` privado hasta sanear | [ADR-0013](adr/0013-public-visibility.md) |

> Las decisiones de esta tabla son de arquitectura/proceso y producto. Cualquier cambio requiere un
> nuevo ADR que superseda al anterior.
