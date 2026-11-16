# ADR-0013 — Visibilidad pública del proyecto

- **Estado:** Aceptado
- **Fecha:** 2026-11-16
- **Decisores:** Geronimo Acevedo
- **Relacionados:** [ADR-0008](0008-repo-governance-and-naming.md), [ADR-0009](0009-product-currency-and-simulation.md), [ADR-0011](0011-platform-and-operations.md), [Configuración de la organización](../governance/org-setup.md), [Seguridad](../operations/security.md)
- **Supera parcialmente:** D-15 del [ADR-0008](0008-repo-governance-and-naming.md) — visibilidad de repositorios.

## Contexto

El [ADR-0008](0008-repo-governance-and-naming.md) decidió repositorios privados al inicio, públicos solo
si el proyecto se usa como portfolio. El proyecto se va a usar como portfolio: su valor está en los
ADRs, el gobierno multi-repositorio y la práctica de banca, no solo en el código.

Mantener todo privado hasta el MVP retrasa ese valor; publicar al final hace perder el historial de
decisiones y de PRs, que es precisamente lo que demuestra criterio.

Publicar es prácticamente irreversible: los forks, las cachés y los índices de búsqueda sobreviven a
un cambio a privado. Por eso la decisión debe decir con precisión qué se publica y qué no.

## Decisión

Publicar desde el inicio los repositorios que muestran ingeniería, y mantener privado el que expone
detalles de infraestructura:

| Repositorio | Visibilidad | Motivo |
| --- | --- | --- |
| `.github` | Público | Gobierno y plantillas visibles. |
| `docs` | Público | ADRs, arquitectura y decisiones. |
| `contracts` | Público | Contratos versionados y compatibilidad. |
| `platform` | Público | Experiencia de desarrollo y Floci. |
| `service-template` | Público | Estándar de servicio y CI. |
| `core-banking-service` | Público | Ledger, idempotencia y tests. |
| `transfers-service` | Público | Outbox, conciliación y contratos. |
| `gitops` | Público | Estado declarativo y promoción por digest. |
| `web` | Público | Aplicación cliente y backoffice. |
| `infra` | **Privado** | Terraform expone cuentas, nombres de recursos y topología. |

Reglas derivadas:

- `infra` permanece privado hasta que no contenga IDs de cuenta, ARNs ni nombres de recursos reales.
  Publicarlo requiere un ADR nuevo que lo autorice.
- Nunca se publican: `.env`, `terraform.tfstate`, `.terraform/`, ARNs reales, IDs de cuenta, capturas
  con credenciales, tokens ni datos personales.
- Los commits nuevos usan el email `noreply` de GitHub. El historial existente no se reescribe:
  hacerlo rompe los PRs ya mezclados y las URLs.
- Los datos son sintéticos, según el [ADR-0009](0009-product-currency-and-simulation.md).
- El perfil de la organización y el README de cada repo declaran que es un proyecto educativo, con
  dinero simulado y sin afiliación a entidades financieras.
- Cuando exista CI, incluye detección de secretos en el historial y en el diff.

## Consecuencias

### Positivas

- El historial de ADRs y PRs queda visible: es la parte que diferencia el portfolio.
- Publicar impone disciplina: lo que se sube no puede contener secretos.
- Los repos públicos tienen Actions sin coste y rulesets disponibles en el plan Free.
- La organización se puede enlazar desde el CV como conjunto, no como un repo suelto.

### Negativas

- Un descuido queda público de forma permanente.
- El email personal aparece en los commits anteriores a este ADR.
- `infra` requiere saneamiento antes de publicarse, y mientras tanto el portfolio muestra una pieza
  menos.

### Mitigaciones

- `infra` privado por defecto y revisión explícita antes de abrirlo.
- Secret scanning y push protection activos en la organización.
- Revisión de capturas, logs y ejemplos antes de publicarlos.
- `.gitignore` con `tfstate`, `.terraform/` y `.env` desde el primer commit de cada repo.

## Alternativas consideradas

1. **Todo privado hasta el MVP.** Se descarta: pierde el historial y el portfolio llega tarde.
2. **Solo `docs` y `contracts` públicos.** Se descarta: muestra intención pero no ejecución.
3. **Todo público, incluido `infra`.** Se descarta: el Terraform filtra topología y nombres de
   cuenta, y ese detalle no aporta nada al portfolio.

## Notas

Este ADR supersede el D-15 del [ADR-0008](0008-repo-governance-and-naming.md). El resto de sus
decisiones sigue vigente.
