---
id: DATA-diccionario
estado: borrador
actualizado: 2026-09-06
---

# Diccionario de datos

Vocabulario canónico español → inglés para nombres de entidad y de campo. Nace de la
auditoría del 2026-09-06 sobre las 54 historias ya escritas (`04-historias/`), que
mezclaban español e inglés de forma inconsistente — cierra esa deuda de una sola vez, antes
de que exista código, y es la referencia que toda historia nueva debe seguir de aquí en
adelante (`AGENTS.md` raíz §5.4: specs en español, código en inglés).

## Cómo se usa esta tabla

- La columna **Español** es el término tal como aparece en `jdlargo-specs` (glosario,
  historias, documento del cliente).
- La columna **Inglés (código)** es el identificador que debe usarse en
  `jdlargo-api`/`jdlargo-web`: nombre de tabla/entidad en `PascalCase` o de campo en
  `snake_case`, según aplique.
- Un campo que no aparezca aquí se traduce de forma directa a inglés, `snake_case`,
  siguiendo el mismo patrón `entidad.campo` que ya usan las historias.
- **Nunca se traduce** un identificador `HU-xxx` / `RF-xxx` / `PA-xxx` / `ADR-xxxx` /
  `CAP-xx` / `EP-xxx`, ni el texto de cara al usuario final (queda en español).

## Advertencia — "organización" significa dos cosas distintas

La palabra española **"organización" tiene dos sentidos que no pueden compartir el mismo
nombre en inglés**, y confundirlos sería el error más caro de este diccionario:

1. **Organización cliente** (el *tenant*, quien contrata el SaaS) → `organization`. Es el
   `organization_id` que ya aparece, correcto, en las 50 historias como llave de
   aislamiento multi-tenant (`HU-002`). **No cambia.**
2. **Organización** como tipo de `Sujeto` (una persona jurídica que es contraparte o
   persona relacionada — está *dentro* de una organización cliente, no es una) →
   `legal_entity`. **Nunca `organization`.**

## Vocabulario canónico

| Español | Inglés (código) | Nota |
|---|---|---|
| Organización cliente / tenant | `organization` | Ya consistente en las 50 historias. Ver advertencia arriba |
| Usuario | `user` | — |
| Membresía | `membership` | `Usuario › Membresía › Organización` (`actores-y-roles.md`) |
| Rol / permiso | `role` / `permission` | — |
| Versión de configuración | `configuration_version` | `ADR-0004` |
| Estándar | `standard` | SARLAFT, SAGRILAFT, PTEE |
| Matriz de requisitos | `requirement_matrix` | — |
| Metodología de riesgo | `risk_methodology` | — |
| Tipo de contraparte | `counterparty_type` | — |
| Catálogo de fuentes / Fuente | `source` | `HU-023` |
| Credencial (de fuente o de integración) | `credential` | `HU-023`, `HU-054` |
| **Sujeto** (supertipo: persona natural u organización, a nivel de organización cliente) | `party` | No usar `subject`: choca con el *claim* `sub` de un JWT y con `Subject` de RxJS en el mismo stack |
| Persona natural | `natural_person` | Subtipo de `party` |
| Organización (persona jurídica, sujeto de DD) | `legal_entity` | Subtipo de `party`. **Nunca `organization`** — ver advertencia |
| Contraparte | `counterparty` | Un `party` en el rol de contraparte de una organización cliente |
| Persona relacionada | `related_party` | Representante legal, beneficiario final, accionista, administrador, apoderado — `HU-029` |
| Relación (arista entre sujetos) | `relationship` | `HU-029` — es de primera clase, no una columna |
| Beneficiario final | `beneficial_owner` | Sigla `UBO` (*ultimate beneficial owner*), término estándar AML — no traducir literal |
| **Expediente** | `dossier` | **Nunca `case`**: ese nombre queda para el caso de alerta (`HU-028`) |
| Solicitud (de vinculación) | `onboarding_request` | `HU-008` |
| Estado (del expediente/documento, como máquina de estados) | `state` | Módulo/transiciones — `HU-009`, `dossiers/states/` en `arquitectura-de-aplicacion.md` |
| Estado (como atributo puntual de un registro) | `status` | Campo, no módulo — p. ej. `document.status` |
| Afirmación | `assertion` | `ADR-0005`, procedencia: `declarado · extraído · verificado · evaluado` |
| Documento / Tipo documental | `document` / `document_type` | — |
| Vigencia (de un documento) | `validity` (o `expires_at` si es una fecha puntual) | `HU-021` |
| Verificación | `verification` | `HU-024` |
| Screening | `screening` | Ya es préstamo del inglés, no se traduce |
| Coincidencia (de screening) | `match` | `HU-026`. Distinta de `alert` — ver `modelo-conceptual.md` |
| Alerta | `alert` | Ver `modelo-conceptual.md`: coincidencia ≠ alerta ≠ caso |
| Caso (de alerta, `HU-027`/`HU-028`) | `case` | — |
| Caso de prueba del simulador de configuración (`HU-039`) | `simulation_case` | **No `test_case`**: choca con pruebas de software y con `case` de alertas |
| Evaluación de riesgo | `risk_assessment` | `HU-032` |
| Debida diligencia intensificada (DDI) | `edd` | *Enhanced due diligence*, sigla estándar AML. La prosa en español sigue diciendo "DDI"; el campo/booleano de código usa `edd` |
| Decisión | `decision` | `HU-015` |
| Condición (de una decisión) | `condition` | — |
| Consentimiento | `consent` | `HU-011` |
| Firma (electrónica) | `signature` | `HU-022` |
| Bitácora | `audit_log` | `HU-006`, `ADR-0007` |
| Ejecución de IA | `ai_execution` | `HU-018` |
| Evento de monitoreo | `monitoring_event` | `HU-040` |
| Renovación | `renewal` | `HU-042` |
| Plan (comercial) | `plan` | `HU-046` |
| Cupo (de consultas) | `quota` | `HU-046`, `HU-047` |
| Ciclo (de facturación) | `billing_cycle` | `HU-048` |
| Excedente | `overage` | `HU-047`, `ADR-0002` |
| Consumo | `consumption` | Ya usado así en `arquitectura-de-aplicacion.md` (`server/consumption/`) |
| Factura | `invoice` | `HU-050` |
| Pago | `payment` | `HU-049` |
| Huella (de archivo o de credencial) | `hash` | `HU-023`, `HU-054` |
| Dirección de red | `ip_address` | — |
| Enlace de acceso (portal de la contraparte) | `access_link` | `HU-010` |

## Las cuatro entidades pendientes de detallar

De `modelo-conceptual.md`: Política, Condición, Tipo documental (ya cubierto arriba),
Modelo de IA. Se completan aquí a medida que se escriban sus historias.

**Sensible** = dato personal o de debida diligencia con tratamiento especial. Marcarlo
importa: define cifrado, retención y quién puede verlo.
