---
id: HIST-backlog
estado: vivo
actualizado: 2026-08-27
---

# Backlog

Índice de épicas e historias. Se actualiza cada vez que se crea o cambia de estado una historia.

**50 historias en 8 épicas**, una épica por fase del `02-producto/roadmap.md`. Todas en estado
`borrador`: la Definition of Ready del repositorio lo impide mientras dependan de una `PA-xxx` sin
responder.

## Épicas

| ID | Épica | Capacidad | Fase | Historias | Estado |
|----|-------|-----------|------|-----------|--------|
| `EP-000` | [Cimientos](EP-000-cimientos/EP-000-cimientos.md) | `CAP-00` | 0 | 6 | borrador |
| `EP-001` | [Un expediente completo, a mano](EP-001-expediente-completo-a-mano/EP-001-expediente-completo-a-mano.md) | `CAP-01` | 1 | 10 | borrador |
| `EP-002` | [El expediente se llena solo](EP-002-el-expediente-se-llena-solo/EP-002-el-expediente-se-llena-solo.md) | `CAP-02` | 2 | 6 | borrador |
| `EP-003` | [El expediente se verifica](EP-003-el-expediente-se-verifica/EP-003-el-expediente-se-verifica.md) | `CAP-03` | 3 | 6 | borrador |
| `EP-004` | [El expediente se califica](EP-004-el-expediente-se-califica/EP-004-el-expediente-se-califica.md) | `CAP-04` | 4 | 5 | borrador |
| `EP-005` | [El cliente se autogestiona](EP-005-el-cliente-se-autogestiona/EP-005-el-cliente-se-autogestiona.md) | `CAP-05` | 5 | 6 | borrador |
| `EP-006` | [El expediente vive](EP-006-el-expediente-vive/EP-006-el-expediente-vive.md) | `CAP-06` | 6 | 6 | borrador |
| `EP-007` | [Capa comercial](EP-007-capa-comercial/EP-007-capa-comercial.md) | `CAP-07` | C | 5 | borrador |

## EP-000 — Cimientos

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-001` | [Organizaciones, cuentas de usuario y pertenencia](EP-000-cimientos/HU-001-organizaciones-cuentas-y-pertenencia.md) | Must | `PA-024` |
| `HU-002` | [Aislamiento entre organizaciones con contexto de usuario](EP-000-cimientos/HU-002-aislamiento-entre-organizaciones.md) | Must | — |
| `HU-003` | [Permisos por rol como configuración de la organización](EP-000-cimientos/HU-003-permisos-por-rol-como-configuracion.md) | Must | `PA-024` |
| `HU-004` | [Publicación de versiones de configuración inmutables](EP-000-cimientos/HU-004-publicacion-de-versiones-de-configuracion.md) | Must | `PA-025` |
| `HU-005` | [Registro de afirmaciones con procedencia](EP-000-cimientos/HU-005-registro-de-afirmaciones-con-procedencia.md) | Must | `PA-027` |
| `HU-006` | [Bitácora inmutable transversal](EP-000-cimientos/HU-006-bitacora-inmutable-transversal.md) | Must | `PA-026`, `PA-009` |

Orden: `HU-001` → `HU-002` → `HU-006` → `HU-004` → `HU-003` → `HU-005`.

## EP-001 — Un expediente completo, a mano

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-007` | [Tipos de contraparte y matriz de requisitos](EP-001-expediente-completo-a-mano/HU-007-tipos-de-contraparte-y-matriz-de-requisitos.md) | Must | `PA-017`, `PA-018`, `PA-001` |
| `HU-008` | [Crear la solicitud de vinculación y abrir el expediente](EP-001-expediente-completo-a-mano/HU-008-crear-solicitud-y-abrir-expediente.md) | Must | `PA-025`, `PA-031` |
| `HU-009` | [Máquina de estados del expediente](EP-001-expediente-completo-a-mano/HU-009-maquina-de-estados-del-expediente.md) | Must | `PA-029`, `PA-028` |
| `HU-010` | [Acceso de la contraparte por enlace](EP-001-expediente-completo-a-mano/HU-010-acceso-de-la-contraparte-por-enlace.md) | Must | `PA-028`, `PA-031`, `PA-019` |
| `HU-011` | [Aviso de privacidad y evidencia del consentimiento](EP-001-expediente-completo-a-mano/HU-011-aviso-de-privacidad-y-consentimiento.md) | Must | `PA-031`, `PA-021` |
| `HU-012` | [Formulario dinámico de identificación](EP-001-expediente-completo-a-mano/HU-012-formulario-dinamico-de-identificacion.md) | Must | `PA-018` |
| `HU-013` | [Carga de los documentos exigidos](EP-001-expediente-completo-a-mano/HU-013-carga-de-los-documentos-exigidos.md) | Must | `PA-030`, `PA-009` |
| `HU-014` | [Revisión del expediente y solicitud de correcciones](EP-001-expediente-completo-a-mano/HU-014-revision-del-expediente-y-correcciones.md) | Must | `PA-028`, `PA-029` |
| `HU-015` | [Decisión del Oficial de Cumplimiento](EP-001-expediente-completo-a-mano/HU-015-decision-del-oficial-de-cumplimiento.md) | Must | `PA-029`, `PA-031` |
| `HU-016` | [Expediente electrónico reconstruible](EP-001-expediente-completo-a-mano/HU-016-expediente-electronico-reconstruible.md) | Must | `PA-009` |

Orden: el del recorrido, de `HU-007` a `HU-016`.

## EP-002 — El expediente se llena solo

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-017` | [Extracción de datos desde los documentos](EP-002-el-expediente-se-llena-solo/HU-017-extraccion-de-datos-desde-los-documentos.md) | Must | `PA-032`, `PA-021` |
| `HU-018` | [Registro de cada ejecución de IA](EP-002-el-expediente-se-llena-solo/HU-018-registro-de-cada-ejecucion-de-ia.md) | Must | `PA-021` |
| `HU-019` | [Conciliación de lo declarado con lo extraído](EP-002-el-expediente-se-llena-solo/HU-019-conciliacion-de-lo-declarado-con-lo-extraido.md) | Must | `PA-027`, `PA-029` |
| `HU-020` | [Validación humana de lo extraído](EP-002-el-expediente-se-llena-solo/HU-020-validacion-humana-de-lo-extraido.md) | Must | `PA-032` |
| `HU-021` | [Vigencias y estados del documento](EP-002-el-expediente-se-llena-solo/HU-021-vigencias-y-estados-del-documento.md) | Must | `PA-018`, `PA-027` |
| `HU-022` | [Firma electrónica de niveles 1 y 2](EP-002-el-expediente-se-llena-solo/HU-022-firma-electronica-niveles-1-y-2.md) | Should | `PA-019`, `PA-020` |

Orden: `HU-018` → `HU-017` → `HU-019` → `HU-020` → `HU-021` → `HU-022`.

## EP-003 — El expediente se verifica

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-023` | [Catálogo de fuentes externas](EP-003-el-expediente-se-verifica/HU-023-catalogo-de-fuentes-externas.md) | Must | `PA-005`, `PA-012` |
| `HU-024` | [Verificación de datos contra fuentes externas](EP-003-el-expediente-se-verifica/HU-024-verificacion-contra-fuentes-externas.md) | Must | `PA-005`, `PA-012` |
| `HU-025` | [Screening contra listas, PEP y sanciones](EP-003-el-expediente-se-verifica/HU-025-screening-contra-listas-pep-y-sanciones.md) | Must | `PA-005`, `PA-012`, `PA-014` |
| `HU-026` | [Comparación de nombres e identificadores](EP-003-el-expediente-se-verifica/HU-026-comparacion-de-nombres-e-identificadores.md) | Must | `PA-033` |
| `HU-027` | [Alertas](EP-003-el-expediente-se-verifica/HU-027-alertas.md) | Must | `PA-031` |
| `HU-028` | [Casos: análisis, decisión y cierre](EP-003-el-expediente-se-verifica/HU-028-casos-analisis-decision-y-cierre.md) | Must | — |

Orden: `HU-023` → `HU-027` → `HU-028` → `HU-024` → `HU-025` → `HU-026`.

## EP-004 — El expediente se califica

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-029` | [Personas relacionadas y grafo de relaciones](EP-004-el-expediente-se-califica/HU-029-personas-relacionadas-y-grafo-de-relaciones.md) | Must | `PA-017`, `PA-018` |
| `HU-030` | [Identificación del beneficiario final](EP-004-el-expediente-se-califica/HU-030-identificacion-del-beneficiario-final.md) | Must | `PA-005`, `PA-018` |
| `HU-031` | [Metodología de riesgo configurable](EP-004-el-expediente-se-califica/HU-031-metodologia-de-riesgo-configurable.md) | Must | `PA-017`, `PA-018` |
| `HU-032` | [Evaluación de riesgo y clasificación final](EP-004-el-expediente-se-califica/HU-032-evaluacion-de-riesgo-y-clasificacion-final.md) | Must | `PA-034` |
| `HU-033` | [Debida diligencia intensificada](EP-004-el-expediente-se-califica/HU-033-debida-diligencia-intensificada.md) | Must | `PA-018`, `PA-029` |

Orden: `HU-031` → `HU-029` → `HU-030` → `HU-032` → `HU-033`.

## EP-005 — El cliente se autogestiona

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-034` | [Administrar versiones de configuración](EP-005-el-cliente-se-autogestiona/HU-034-administrar-versiones-de-configuracion.md) | Must | `PA-017`, `PA-025` |
| `HU-035` | [Administrar tipos de contraparte y matriz de requisitos](EP-005-el-cliente-se-autogestiona/HU-035-administrar-tipos-de-contraparte-y-matriz.md) | Must | `PA-017`, `PA-018` |
| `HU-036` | [Administrar la metodología de riesgo](EP-005-el-cliente-se-autogestiona/HU-036-administrar-la-metodologia-de-riesgo.md) | Must | `PA-017`, `PA-034` |
| `HU-037` | [Administrar fuentes y aviso de privacidad](EP-005-el-cliente-se-autogestiona/HU-037-administrar-fuentes-y-aviso-de-privacidad.md) | Should | `PA-005`, `PA-012`, `PA-021` |
| `HU-038` | [Administrar usuarios, roles y permisos](EP-005-el-cliente-se-autogestiona/HU-038-administrar-usuarios-roles-y-permisos.md) | Must | `PA-024`, `PA-031` |
| `HU-039` | [Probar la configuración antes de publicar](EP-005-el-cliente-se-autogestiona/HU-039-probar-la-configuracion-antes-de-publicar.md) | Should | `PA-017` |

Orden: `HU-034` → `HU-035` → `HU-039` → `HU-036` → `HU-038` → `HU-037`.

## EP-006 — El expediente vive

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-040` | [Monitoreo continuo](EP-006-el-expediente-vive/HU-040-monitoreo-continuo.md) | Must | `PA-035`, `PA-014`, `PA-005` |
| `HU-041` | [Vencimientos y avisos anticipados](EP-006-el-expediente-vive/HU-041-vencimientos-y-avisos-anticipados.md) | Must | `PA-031` |
| `HU-042` | [Renovación y actualización periódica](EP-006-el-expediente-vive/HU-042-renovacion-y-actualizacion-periodica.md) | Must | `PA-035`, `PA-025` |
| `HU-043` | [Panel del Oficial de Cumplimiento](EP-006-el-expediente-vive/HU-043-panel-del-oficial-de-cumplimiento.md) | Must | `PA-039` |
| `HU-044` | [Exportación del expediente y de reportes](EP-006-el-expediente-vive/HU-044-exportacion-del-expediente-y-de-reportes.md) | Should | `PA-009` |
| `HU-045` | [Recordatorios a la contraparte](EP-006-el-expediente-vive/HU-045-recordatorios-a-la-contraparte.md) | Could | `PA-031`, `PA-028`, `PA-019` |

Orden: `HU-041` → `HU-040` → `HU-042` → `HU-043` → `HU-044` → `HU-045`.

## EP-007 — Capa comercial

| ID | Historia | Prioridad | Bloqueada por |
|----|----------|-----------|---------------|
| `HU-046` | [Planes, licencias y cupo de consultas](EP-007-capa-comercial/HU-046-planes-licencias-y-cupo-de-consultas.md) | Must | `PA-036`, `PA-038` |
| `HU-047` | [Medición de consumo y control del cupo](EP-007-capa-comercial/HU-047-medicion-de-consumo-y-control-del-cupo.md) | Must | `PA-037`, `PA-036` |
| `HU-048` | [Cierre de ciclo y cálculo del excedente](EP-007-capa-comercial/HU-048-cierre-de-ciclo-y-calculo-del-excedente.md) | Must | `PA-036`, `PA-037` |
| `HU-049` | [Cobro por pasarela](EP-007-capa-comercial/HU-049-cobro-por-pasarela.md) | Must | **`PA-016`**, `PA-036` |
| `HU-050` | [Factura electrónica](EP-007-capa-comercial/HU-050-factura-electronica.md) | Must | `PA-036`, `PA-038` |

Orden: el de la lista.

## Preguntas abiertas que bloquean el backlog

| Pregunta | Historias que bloquea | Impacto |
|---|---|---|
| `PA-016` | `HU-049` | **Bloqueante.** Si el cobro desatendido solo funciona con una marca de tarjeta, los clientes pequeños no son cobrables automáticamente. Es una llamada, no una decisión de diseño |
| `PA-025` | `HU-004`, `HU-008`, `HU-034`, `HU-042` | **Alto.** Define si el expediente congela su versión de configuración al abrirse o al evaluarse |
| `PA-026` | `HU-006` | **Alto.** Define si la bitácora es solo inserción o exige protección criptográfica |
| `PA-029` | `HU-009`, `HU-014`, `HU-015`, `HU-019`, `HU-033` | **Alto.** Define si se puede decidir con requisitos pendientes |
| `PA-033` | `HU-026` | **Alto.** El umbral de matching decide entre inundar de falsos positivos o dejar pasar coincidencias |
| `PA-035` | `HU-040`, `HU-042` | **Alto.** La frecuencia del monitoreo es la principal fuente de costo variable |
| `PA-036` | toda `EP-007` | **Bloqueante para la capa comercial.** Planes, precios y cupos |
| `PA-017` | `HU-007`, `HU-029`, `HU-031`, toda `EP-005` | **Alto sobre el esfuerzo.** Puede reducir `EP-005` a la mitad o duplicarla |
| `PA-005` y `PA-012` | `EP-003`, `HU-030`, `HU-037`, `HU-040` | Qué fuentes existen y cuánto cuestan. Sin ellas no hay estimación de costo |
| `PA-014` | `HU-025`, `HU-040`, `EP-007` | Cuántas contrapartes tiene un cliente típico: el otro factor del costo |
| `PA-024` | `HU-001`, `HU-003`, `HU-038` | Si el rol es configurable o un catálogo común |
| `PA-032` | `HU-017`, `HU-020` | Umbral de confianza que obliga a validación humana |
| `PA-034` | `HU-032`, `HU-036` | Si la clasificación de riesgo exige segunda aprobación |
| `PA-039` | `HU-043` | Qué indicadores quiere el Oficial en su panel |
| `PA-009` | `HU-006`, `HU-013`, `HU-016`, `HU-044` | Retención; no bloquea estructura |
| `PA-018` | Varias de `EP-001` a `EP-005` | Volumen de configuración inicial |
| `PA-019`, `PA-020` | `HU-010`, `HU-022`, `HU-045` | Segundo factor por mensaje de texto y firma certificada |
| `PA-021` | `HU-011`, `HU-017`, `HU-018`, `HU-037` | Proveedores de IA y transferencia internacional de datos |
| `PA-027` | `HU-005`, `HU-019`, `HU-021` | Precedencia entre orígenes de una afirmación |
| `PA-028` | `HU-009`, `HU-010`, `HU-014`, `HU-045` | Qué pasa si el enlace expira sin completarse |
| `PA-030` | `HU-013` | Formatos y tamaño máximo de documentos |
| `PA-031` | `HU-008`, `HU-010`, `HU-011`, `HU-015`, `HU-027`, `HU-038`, `HU-041`, `HU-045` | Qué notifica la plataforma y a quién |
| `PA-037`, `PA-038` | `EP-007` | Comportamiento al agotar el cupo y alta de clientes |
| `PA-001` | `HU-007` | Qué estándares entran en la configuración |

**Las tres que más urgen**, porque tocan el modelo de datos de la Fase 0 y no la interfaz:
`PA-025`, `PA-026` y `PA-029`.

## Cobertura del criterio de aceptación del cliente (§44)

| Pregunta de la §44 | Épica que la contesta |
|---|---|
| A · Quién era la contraparte | `EP-001` |
| B · Qué información entregó | `EP-001` |
| C · Qué documentos presentó | `EP-001` |
| D · Qué información fue extraída automáticamente | `EP-002` |
| E · Qué información fue verificada | `EP-003` |
| F · Qué fuentes fueron consultadas | `EP-003` |
| G · Qué alertas aparecieron | `EP-003` |
| H · Quién analizó las alertas | `EP-003` |
| I · Qué metodología de riesgo se aplicó | `EP-004` |
| J · Qué nivel de riesgo resultó | `EP-004` |
| K · Si hubo debida diligencia intensificada | `EP-004` |
| L · Quién tomó la decisión | `EP-001` |
| M · Por qué la tomó | `EP-001` |
| N · Qué condiciones quedaron | `EP-001` |
| O · Cuándo debe actualizarse | `EP-006` |
| P · Qué ocurrió durante el monitoreo | `EP-006` |

Al cerrar `EP-006` están las dieciséis: es el producto terminado según el criterio del propio
cliente. `EP-005` y `EP-007` no añaden respuestas a la §44 —añaden autonomía del cliente y
capacidad de cobrar.
