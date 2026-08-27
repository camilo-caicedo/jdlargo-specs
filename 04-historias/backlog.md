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
| `EP-001` | [Un expediente completo, a mano](EP-001-expediente-completo-a-mano/EP-001-expediente-completo-a-mano.md) | `CAP-01` | 1 | 10 | borrador |

## Historias

### EP-000 — Cimientos

| ID | Historia | Prioridad | Bloqueada por | Estado |
|----|----------|-----------|---------------|--------|
| `HU-001` | [Organizaciones, cuentas de usuario y pertenencia](EP-000-cimientos/HU-001-organizaciones-cuentas-y-pertenencia.md) | Must | `PA-024` | borrador |
| `HU-002` | [Aislamiento entre organizaciones con contexto de usuario](EP-000-cimientos/HU-002-aislamiento-entre-organizaciones.md) | Must | — (depende de `HU-001`) | borrador |
| `HU-003` | [Permisos por rol como configuración de la organización](EP-000-cimientos/HU-003-permisos-por-rol-como-configuracion.md) | Must | `PA-024` | borrador |
| `HU-004` | [Publicación de versiones de configuración inmutables](EP-000-cimientos/HU-004-publicacion-de-versiones-de-configuracion.md) | Must | `PA-025` | borrador |
| `HU-005` | [Registro de afirmaciones con procedencia](EP-000-cimientos/HU-005-registro-de-afirmaciones-con-procedencia.md) | Must | `PA-027` (no bloquea el diseño) | borrador |
| `HU-006` | [Bitácora inmutable transversal](EP-000-cimientos/HU-006-bitacora-inmutable-transversal.md) | Must | `PA-026`, `PA-009` | borrador |

Orden de construcción: `HU-001` → `HU-002` → `HU-006` → `HU-004` → `HU-003` → `HU-005`.

La bitácora se adelanta a propósito: todas las historias posteriores escriben en ella, y una
bitácora retroajustada deja un tramo del historial sin explicación. El versionamiento va antes
que los permisos porque la matriz de permisos es, por `ADR-0004`, una configuración versionada
más.

### EP-001 — Un expediente completo, a mano

| ID | Historia | Prioridad | Bloqueada por | Estado |
|----|----------|-----------|---------------|--------|
| `HU-007` | [Tipos de contraparte y matriz de requisitos](EP-001-expediente-completo-a-mano/HU-007-tipos-de-contraparte-y-matriz-de-requisitos.md) | Must | `PA-017`, `PA-018`, `PA-001` | borrador |
| `HU-008` | [Crear la solicitud de vinculación y abrir el expediente](EP-001-expediente-completo-a-mano/HU-008-crear-solicitud-y-abrir-expediente.md) | Must | `PA-025`, `PA-031` | borrador |
| `HU-009` | [Máquina de estados del expediente](EP-001-expediente-completo-a-mano/HU-009-maquina-de-estados-del-expediente.md) | Must | `PA-029`, `PA-028` | borrador |
| `HU-010` | [Acceso de la contraparte por enlace](EP-001-expediente-completo-a-mano/HU-010-acceso-de-la-contraparte-por-enlace.md) | Must | `PA-028`, `PA-031`, `PA-019` | borrador |
| `HU-011` | [Aviso de privacidad y evidencia del consentimiento](EP-001-expediente-completo-a-mano/HU-011-aviso-de-privacidad-y-consentimiento.md) | Must | `PA-031`, `PA-021` | borrador |
| `HU-012` | [Formulario dinámico de identificación](EP-001-expediente-completo-a-mano/HU-012-formulario-dinamico-de-identificacion.md) | Must | `PA-018` | borrador |
| `HU-013` | [Carga de los documentos exigidos](EP-001-expediente-completo-a-mano/HU-013-carga-de-los-documentos-exigidos.md) | Must | `PA-030`, `PA-009` | borrador |
| `HU-014` | [Revisión del expediente y solicitud de correcciones](EP-001-expediente-completo-a-mano/HU-014-revision-del-expediente-y-correcciones.md) | Must | `PA-028`, `PA-029` | borrador |
| `HU-015` | [Decisión del Oficial de Cumplimiento](EP-001-expediente-completo-a-mano/HU-015-decision-del-oficial-de-cumplimiento.md) | Must | `PA-029`, `PA-031` | borrador |
| `HU-016` | [Expediente electrónico reconstruible](EP-001-expediente-completo-a-mano/HU-016-expediente-electronico-reconstruible.md) | Must | `PA-009` | borrador |

Orden de construcción: el del recorrido, de `HU-007` a `HU-016`. Coincide con la §45 del
cliente: primero el modelo de cumplimiento, la pantalla al final.

## Estado general

Ninguna historia pasa de `borrador`. La Definition of Ready del repo lo impide mientras dependan
de una `PA-xxx` sin responder.

| Pregunta | Bloquea | Impacto |
|---|---|---|
| `PA-001` | `HU-007` | Qué estándares existen y entran en la configuración |
| `PA-009` | `HU-006`, `HU-013`, `HU-016` | Retención; no bloquea la estructura |
| `PA-017` | `HU-007` | Quién configura la matriz en la práctica |
| `PA-018` | `HU-007`, `HU-012` | Cuántos estándares y tipos de contraparte hay que soportar |
| `PA-019` | `HU-010` | Si el segundo factor por mensaje de texto es necesario |
| `PA-021` | `HU-011` | Qué proveedores de IA se declaran en el aviso de privacidad |
| `PA-024` | `HU-001`, `HU-003` | Si el rol es configurable por organización cliente o un catálogo común |
| `PA-025` | `HU-004`, `HU-008` | Si el expediente congela la versión al abrirse o al evaluarse |
| `PA-026` | `HU-006` | Si la bitácora es una tabla de solo inserción o exige protección criptográfica |
| `PA-027` | `HU-005` | No bloquea el diseño: la precedencia se calcula en la Fase 2 |
| `PA-028` | `HU-009`, `HU-010`, `HU-014` | Qué pasa si el enlace de acceso expira sin completarse |
| `PA-029` | `HU-009`, `HU-014`, `HU-015` | Si se puede decidir con requisitos pendientes |
| `PA-030` | `HU-013` | Formatos y tamaño máximo de documentos |
| `PA-031` | `HU-008`, `HU-010`, `HU-011`, `HU-015` | Quién entrega el enlace y qué se notifica |

Las tres de mayor impacto son `PA-025`, `PA-026` y `PA-029`: cambian el modelo de datos, no la
interfaz, y elegir mal se paga rehaciendo la tabla y todo lo que ya escribió en ella.

## Cobertura del criterio de aceptación del cliente (§44)

| Pregunta de la §44 | Épica que la contesta |
|---|---|
| A · Quién era la contraparte | `EP-001` |
| B · Qué información entregó | `EP-001` |
| C · Qué documentos presentó | `EP-001` |
| D · Qué se extrajo automáticamente | Fase 2 |
| E · Qué se verificó · F · Qué fuentes · G · Qué alertas · H · Quién las analizó | Fase 3 |
| I · Qué metodología · J · Qué nivel de riesgo · K · Si hubo debida diligencia intensificada | Fase 4 |
| L · Quién decidió · M · Por qué · N · Qué condiciones | `EP-001` |
| O · Cuándo actualizar · P · Qué pasó en el monitoreo | Fase 6 |

## Épicas por abrir

| Fase | Épica prevista | Estado |
|---|---|---|
| 2 | El expediente se llena solo | Sin abrir |
| 3 | El expediente se verifica | Sin abrir |
| 4 | El expediente se califica | Sin abrir |
| 5 | El cliente se autogestiona | Sin abrir |
| 6 | El expediente vive | Sin abrir |
| C | Capa comercial | Sin abrir |

Ver `02-producto/roadmap.md`.
