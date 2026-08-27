---
id: HIST-backlog
estado: vivo
actualizado: 2026-08-27
---

# Backlog

Índice de épicas e historias. Se actualiza cada vez que se crea o cambia de estado una historia.

## Épicas

| ID | Épica | Capacidad | Fase | Historias | Estado |
|----|-------|-----------|------|-----------|--------|
| `EP-000` | [Cimientos](EP-000-cimientos/EP-000-cimientos.md) | `CAP-00` | 0 | 6 | borrador |

## Historias

| ID | Historia | Épica | Prioridad | Bloqueada por | Estado |
|----|----------|-------|-----------|---------------|--------|
| `HU-001` | [Organizaciones, cuentas de usuario y pertenencia](EP-000-cimientos/HU-001-organizaciones-cuentas-y-pertenencia.md) | `EP-000` | Must | `PA-024` | borrador |
| `HU-002` | [Aislamiento entre organizaciones con contexto de usuario](EP-000-cimientos/HU-002-aislamiento-entre-organizaciones.md) | `EP-000` | Must | — (depende de `HU-001`) | borrador |
| `HU-003` | [Permisos por rol como configuración de la organización](EP-000-cimientos/HU-003-permisos-por-rol-como-configuracion.md) | `EP-000` | Must | `PA-024` | borrador |
| `HU-004` | [Publicación de versiones de configuración inmutables](EP-000-cimientos/HU-004-publicacion-de-versiones-de-configuracion.md) | `EP-000` | Must | `PA-025` | borrador |
| `HU-005` | [Registro de afirmaciones con procedencia](EP-000-cimientos/HU-005-registro-de-afirmaciones-con-procedencia.md) | `EP-000` | Must | `PA-027` (no bloquea el diseño) | borrador |
| `HU-006` | [Bitácora inmutable transversal](EP-000-cimientos/HU-006-bitacora-inmutable-transversal.md) | `EP-000` | Must | `PA-026`, `PA-009` | borrador |

## Orden de construcción sugerido

`HU-001` → `HU-002` → `HU-006` → `HU-004` → `HU-003` → `HU-005`

La bitácora (`HU-006`) se adelanta a propósito: todas las historias posteriores escriben en
ella, y una bitácora retroajustada deja un tramo del historial sin explicación. El
versionamiento (`HU-004`) va antes que los permisos (`HU-003`) porque la matriz de permisos es,
por `ADR-0004`, una configuración versionada más.

## Estado general

Ninguna historia pasa de `borrador`. La Definition of Ready del repo lo impide mientras
dependan de una `PA-xxx` sin responder. Pendientes de cerrar para desbloquear `EP-000`:

| Pregunta | Bloquea | Impacto |
|---|---|---|
| `PA-024` | `HU-001`, `HU-003` | Define si el rol es configurable por organización cliente o un catálogo común |
| `PA-025` | `HU-004` | Define si el expediente congela la versión al abrirse o al evaluarse |
| `PA-026` | `HU-006` | Define si la bitácora es una tabla de solo inserción o exige protección criptográfica |
| `PA-027` | `HU-005` | No bloquea el diseño: la precedencia se calcula en la Fase 2 |
| `PA-009` | `HU-006` | Retención; no bloquea la estructura |

## Épicas por abrir

| Fase | Épica prevista | Estado |
|---|---|---|
| 1 | Un expediente completo, a mano | Sin abrir |
| 2 | El expediente se llena solo | Sin abrir |
| 3 | El expediente se verifica | Sin abrir |
| 4 | El expediente se califica | Sin abrir |
| 5 | El cliente se autogestiona | Sin abrir |
| 6 | El expediente vive | Sin abrir |
| C | Capa comercial | Sin abrir |

Ver `02-producto/roadmap.md`.
