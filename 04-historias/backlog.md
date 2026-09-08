---
id: HIST-backlog
estado: vivo
actualizado: 2026-09-06
---

# Backlog

Índice de épicas e historias. Se actualiza cada vez que se crea o cambia de estado una historia.

**60 historias en 10 épicas**, una épica por fase del `02-producto/roadmap.md` (`EP-008` es el
"MVP4" y `EP-009` es presencia pública, ambas fuera de la escalera de fases numeradas).

> **Actualización 2026-09-07 (3).** Se agrega `HU-060` (`RF-060`, `EP-001`) — registro público
> y creación de organización. `PA-038` decía "manual al principio, autoservicio después";
> Camilo decidió adelantar el mecanismo al MVP actual. La capa comercial que lo rodearía
> (planes, cobro, KYB) sigue bloqueada por `PA-043`, sin cambios.

> **Actualización 2026-09-07.** Se agregan `HU-055` a `HU-057` (`RF-055` a `RF-057`, `CAP-01`)
> — inicio de sesión, invitación de miembros y recuperación de contraseña. No son historias del
> descubrimiento original: son un vacío detectado al auditar el avance de `EP-001`. `HU-001`
> excluyó explícitamente pantallas y el flujo de autenticación de la Fase 0, y ninguna historia
> posterior lo recogió — el código ya asumía un usuario autenticado desde `HU-008`. Van en
> `EP-001`, no en `EP-000`, porque son interfaz y `EP-000` se define como verificable sin ella.
>
> **Actualización 2026-09-07 (2).** Se agrega `EP-009` (`CAP-09`, presencia pública) con
> `HU-058` (`RF-058`) — página de inicio pública. Tampoco es del descubrimiento original: no
> existía ninguna capacidad para una superficie pública fuera del portal de la contraparte.

> **Actualización 2026-09-05.** El cliente respondió las 39 preguntas abiertas. **Ninguna
> historia sigue bloqueada por una pregunta sin responder**, salvo las que dependen de las
> derivadas `PA-040` a `PA-045`. La columna "Bloqueada por" pasa a decir **dónde quedó
> resuelta** cada dependencia, que es lo que hace falta para escribirla.
>
> **Actualización 2026-09-06.** Cerrado el otro requisito de la *Definition of Ready* que
> faltaba: la trazabilidad a `RF-xxx` (`03-requisitos/funcionales.md`, `RF-001` a `RF-050`).
> **38 de las 50 historias pasan a `en-revision`.** Las 12 que siguen en `borrador` son las que
> dependen *directamente* de una pregunta derivada todavía abierta:
>
> | Depende de | Historias |
> |---|---|
> | `PA-040` (costo real de fuentes colombianas) | `HU-023`, `HU-024`, `HU-025`, `HU-030`, `HU-037`, `HU-040` |
> | `PA-042` (orden formulario/documentos) | `HU-012` |
> | `PA-043` (precios y cupos definitivos) | `HU-046`, `HU-047`, `HU-048`, `HU-049`, `HU-050` |
>
> **Actualización 2026-09-07.** `PA-042` se resolvió (ver `preguntas-abiertas.md`): sí habrá
> autollenado desde documentos, pero es alcance de `EP-002`, no de `HU-012` — `HU-012` se
> construye manual y **pasa a `en-revision`**. Quedan 11 historias en `borrador`, todas por
> `PA-040`/`PA-043`. La derivada nueva `PA-050` (detalle de la interfaz de confirmación) no
> bloquea nada — se resuelve al arrancar `EP-002`.
>
> `en-revision` no es `aprobado`: falta que alguien —Juan David o quien defina el equipo—
> revise cada una antes de darle luz verde a construirla.
>
> **Actualización 2026-09-06 (2).** Se escribe `EP-008` (MVP4, `CAP-08`) con sus 4 historias
> (`HU-051` a `HU-054`) y sus `RF-051` a `RF-054`. Las cuatro se quedan en `borrador`: todas
> dependen de `PA-046` (qué medio de salida construir), y además `HU-051` de `PA-048` y
> `HU-052` de `PA-047`. Ninguna pasa a `en-revision` todavía.

## Épicas

| ID | Épica | Capacidad | Fase | Historias | Estado |
|----|-------|-----------|------|-----------|--------|
| `EP-000` | [Cimientos](EP-000-cimientos/EP-000-cimientos.md) | `CAP-00` | 0 | 6 | en-revision |
| `EP-001` | [Un expediente completo, a mano](EP-001-expediente-completo-a-mano/EP-001-expediente-completo-a-mano.md) | `CAP-01` | 1 | 14 | borrador |
| `EP-002` | [El expediente se llena solo](EP-002-el-expediente-se-llena-solo/EP-002-el-expediente-se-llena-solo.md) | `CAP-02` | 2 | 6 | en-revision |
| `EP-003` | [El expediente se verifica](EP-003-el-expediente-se-verifica/EP-003-el-expediente-se-verifica.md) | `CAP-03` | 3 | 6 | borrador |
| `EP-004` | [El expediente se califica](EP-004-el-expediente-se-califica/EP-004-el-expediente-se-califica.md) | `CAP-04` | 4 | 5 | borrador |
| `EP-005` | [El cliente se autogestiona](EP-005-el-cliente-se-autogestiona/EP-005-el-cliente-se-autogestiona.md) | `CAP-05` | 5 | 6 | borrador |
| `EP-006` | [El expediente vive](EP-006-el-expediente-vive/EP-006-el-expediente-vive.md) | `CAP-06` | 6 | 6 | borrador |
| `EP-007` | [Capa comercial](EP-007-capa-comercial/EP-007-capa-comercial.md) | `CAP-07` | C | 5 | borrador |
| `EP-008` | [Salida de datos hacia otros sistemas](EP-008-salida-de-datos-hacia-otros-sistemas/EP-008-salida-de-datos-hacia-otros-sistemas.md) | `CAP-08` | MVP4 | 4 | borrador |
| `EP-009` | [Presencia pública](EP-009-presencia-publica/EP-009-presencia-publica.md) | `CAP-09` | — | 2 | borrador |

## EP-000 — Cimientos

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-001` | [Organizaciones, cuentas de usuario y pertenencia](EP-000-cimientos/HU-001-organizaciones-cuentas-y-pertenencia.md) | Must | `PA-024` ✅ `actores-y-roles.md` — identidad global + membresía |
| `HU-002` | [Aislamiento entre organizaciones con contexto de usuario](EP-000-cimientos/HU-002-aislamiento-entre-organizaciones.md) | Must | — |
| `HU-003` | [Permisos por rol como configuración de la organización](EP-000-cimientos/HU-003-permisos-por-rol-como-configuracion.md) | Must | `PA-024` ✅ roles base como plantillas + roles propios por permisos granulares |
| `HU-004` | [Publicación de versiones de configuración inmutables](EP-000-cimientos/HU-004-publicacion-de-versiones-de-configuracion.md) | Must | `PA-025` ✅ `ADR-0004` §2b — congela al abrir y al evaluar |
| `HU-005` | [Registro de afirmaciones con procedencia](EP-000-cimientos/HU-005-registro-de-afirmaciones-con-procedencia.md) | Must | `PA-027` ✅ `ADR-0005` §2 — escala de precedencia configurable |
| `HU-006` | [Bitácora inmutable transversal](EP-000-cimientos/HU-006-bitacora-inmutable-transversal.md) | Must | `PA-026`, `PA-009` ✅ `ADR-0007` — append-only + hash por evento |

Orden: `HU-001` → `HU-002` → `HU-006` → `HU-004` → `HU-003` → `HU-005`.

## EP-001 — Un expediente completo, a mano

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-007` | [Tipos de contraparte y matriz de requisitos](EP-001-expediente-completo-a-mano/HU-007-tipos-de-contraparte-y-matriz-de-requisitos.md) | Must | `PA-017`, `PA-018`, `PA-001` ✅ `ADR-0004` §6 y `vision.md` |
| `HU-008` | [Crear la solicitud de vinculación y abrir el expediente](EP-001-expediente-completo-a-mano/HU-008-crear-solicitud-y-abrir-expediente.md) | Must | `PA-025` ✅ `ADR-0004` §2b · `PA-031` ✅ envío por correo y copia manual |
| `HU-009` | [Máquina de estados del expediente](EP-001-expediente-completo-a-mano/HU-009-maquina-de-estados-del-expediente.md) | Must | `PA-029` ✅ criticidad del requisito · `PA-028` ✅ estado `Expirado/Pendiente` |
| `HU-010` | [Acceso de la contraparte por enlace](EP-001-expediente-completo-a-mano/HU-010-acceso-de-la-contraparte-por-enlace.md) | Must | `PA-028`, `PA-031`, `PA-019` ✅ `ADR-0010` y `arquitectura-de-aplicacion.md` |
| `HU-011` | [Aviso de privacidad y evidencia del consentimiento](EP-001-expediente-completo-a-mano/HU-011-aviso-de-privacidad-y-consentimiento.md) | Must | `PA-031` ✅ · `PA-021` ✅ `RNF-016` a `RNF-018`. Pendiente derivada: `PA-045` |
| `HU-012` | [Formulario dinámico de identificación](EP-001-expediente-completo-a-mano/HU-012-formulario-dinamico-de-identificacion.md) | Must | `PA-018` ✅ `ADR-0004` §6 · `PA-042` ✅ (manual, autollenado es de `EP-002`) |
| `HU-013` | [Carga de los documentos exigidos](EP-001-expediente-completo-a-mano/HU-013-carga-de-los-documentos-exigidos.md) | Must | `PA-030` ✅ `RNF-023` a `RNF-025` · `PA-009` ✅ `ADR-0007` |
| `HU-014` | [Revisión del expediente y solicitud de correcciones](EP-001-expediente-completo-a-mano/HU-014-revision-del-expediente-y-correcciones.md) | Must | `PA-028`, `PA-029` ✅ |
| `HU-015` | [Decisión del Oficial de Cumplimiento](EP-001-expediente-completo-a-mano/HU-015-decision-del-oficial-de-cumplimiento.md) | Must | `PA-029` ✅ decisión excepcional con motivo · `PA-031` ✅ |
| `HU-016` | [Expediente electrónico reconstruible](EP-001-expediente-completo-a-mano/HU-016-expediente-electronico-reconstruible.md) | Must | `PA-009` ✅ `ADR-0007` · salida = Informe de DD (`ADR-0006`) |
| `HU-055` | [Inicio de sesión y selección de organización](EP-001-expediente-completo-a-mano/HU-055-inicio-de-sesion-y-seleccion-de-organizacion.md) | Must | — (vacío detectado 2026-09-07, no depende de ninguna `PA-xxx`) |
| `HU-056` | [Invitar miembros a la organización](EP-001-expediente-completo-a-mano/HU-056-invitar-miembros-a-la-organizacion.md) | Must | — |
| `HU-057` | [Recuperar contraseña olvidada](EP-001-expediente-completo-a-mano/HU-057-recuperar-contrasena-olvidada.md) | Must | — |
| `HU-060` | [Registro público y creación de organización](EP-001-expediente-completo-a-mano/HU-060-registro-publico-y-creacion-de-organizacion.md) | Must | `PA-038` ✅ mecanismo adelantado al MVP 2026-09-07 (decisión de Camilo) |

Orden: el del recorrido, de `HU-007` a `HU-016`. `HU-055` a `HU-057` son transversales a la
épica (autenticación de la app interna) y van **antes** de `HU-010` en el orden real de
construcción.

## EP-002 — El expediente se llena solo

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-017` | [Extracción de datos desde los documentos](EP-002-el-expediente-se-llena-solo/HU-017-extraccion-de-datos-desde-los-documentos.md) | Must | `PA-032` ✅ `ADR-0005` §4b · `PA-021` ✅. Pendiente derivada: `PA-045` |
| `HU-018` | [Registro de cada ejecución de IA](EP-002-el-expediente-se-llena-solo/HU-018-registro-de-cada-ejecucion-de-ia.md) | Must | `PA-021` ✅ router de IA por tenant. Pendiente derivada: `PA-045` |
| `HU-019` | [Conciliación de lo declarado con lo extraído](EP-002-el-expediente-se-llena-solo/HU-019-conciliacion-de-lo-declarado-con-lo-extraido.md) | Must | `PA-027`, `PA-029` ✅ |
| `HU-020` | [Validación humana de lo extraído](EP-002-el-expediente-se-llena-solo/HU-020-validacion-humana-de-lo-extraido.md) | Must | `PA-032` ✅ `ADR-0005` §4b — la decisión es humana |
| `HU-021` | [Vigencias y estados del documento](EP-002-el-expediente-se-llena-solo/HU-021-vigencias-y-estados-del-documento.md) | Must | `PA-018`, `PA-027` ✅ |
| `HU-022` | [Firma electrónica de niveles 1 y 2](EP-002-el-expediente-se-llena-solo/HU-022-firma-electronica-niveles-1-y-2.md) | Should | `PA-019`, `PA-020` ✅ `ADR-0010`. Pendiente derivada: `PA-041` |

Orden: `HU-018` → `HU-017` → `HU-019` → `HU-020` → `HU-021` → `HU-022`.

## EP-003 — El expediente se verifica

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-023` | [Catálogo de fuentes externas](EP-003-el-expediente-se-verifica/HU-023-catalogo-de-fuentes-externas.md) | Must | `PA-005` ✅ catálogo en `07-integraciones/`. **Pendiente: `PA-040`** |
| `HU-024` | [Verificación de datos contra fuentes externas](EP-003-el-expediente-se-verifica/HU-024-verificacion-contra-fuentes-externas.md) | Must | `PA-005` ✅. **Pendiente: `PA-040`** (vía de conexión) |
| `HU-025` | [Screening contra listas, PEP y sanciones](EP-003-el-expediente-se-verifica/HU-025-screening-contra-listas-pep-y-sanciones.md) | Must | `PA-005`, `PA-014` ✅ `ADR-0009`. **Pendiente: `PA-040`** |
| `HU-026` | [Comparación de nombres e identificadores](EP-003-el-expediente-se-verifica/HU-026-comparacion-de-nombres-e-identificadores.md) | Must | `PA-033` ✅ `ADR-0008` — matching multicriterio, tres zonas |
| `HU-027` | [Alertas](EP-003-el-expediente-se-verifica/HU-027-alertas.md) | Must | `PA-031` ✅ |
| `HU-028` | [Casos: análisis, decisión y cierre](EP-003-el-expediente-se-verifica/HU-028-casos-analisis-decision-y-cierre.md) | Must | — |

Orden: `HU-023` → `HU-027` → `HU-028` → `HU-024` → `HU-025` → `HU-026`.

## EP-004 — El expediente se califica

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-029` | [Personas relacionadas y grafo de relaciones](EP-004-el-expediente-se-califica/HU-029-personas-relacionadas-y-grafo-de-relaciones.md) | Must | `PA-017`, `PA-018` ✅ `ADR-0004` §6 |
| `HU-030` | [Identificación del beneficiario final](EP-004-el-expediente-se-califica/HU-030-identificacion-del-beneficiario-final.md) | Must | `PA-005`, `PA-018` ✅. **Pendiente: `PA-040`** (RUES) |
| `HU-031` | [Metodología de riesgo configurable](EP-004-el-expediente-se-califica/HU-031-metodologia-de-riesgo-configurable.md) | Must | `PA-017`, `PA-018` ✅ · `PA-006` ✅ metodología base parametrizable |
| `HU-032` | [Evaluación de riesgo y clasificación final](EP-004-el-expediente-se-califica/HU-032-evaluacion-de-riesgo-y-clasificacion-final.md) | Must | `PA-034` ✅ maker-checker configurable; obligatorio en riesgo alto, PEP y DDI |
| `HU-033` | [Debida diligencia intensificada](EP-004-el-expediente-se-califica/HU-033-debida-diligencia-intensificada.md) | Must | `PA-018`, `PA-029` ✅ |

Orden: `HU-031` → `HU-029` → `HU-030` → `HU-032` → `HU-033`.

## EP-005 — El cliente se autogestiona

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-034` | [Administrar versiones de configuración](EP-005-el-cliente-se-autogestiona/HU-034-administrar-versiones-de-configuracion.md) | Must | `PA-017`, `PA-025` ✅ `ADR-0004` |
| `HU-035` | [Administrar tipos de contraparte y matriz de requisitos](EP-005-el-cliente-se-autogestiona/HU-035-administrar-tipos-de-contraparte-y-matriz.md) | Must | `PA-017`, `PA-018` ✅ — plantillas base + duplicar + versionar |
| `HU-036` | [Administrar la metodología de riesgo](EP-005-el-cliente-se-autogestiona/HU-036-administrar-la-metodologia-de-riesgo.md) | Must | `PA-017`, `PA-034` ✅ |
| `HU-037` | [Administrar fuentes y aviso de privacidad](EP-005-el-cliente-se-autogestiona/HU-037-administrar-fuentes-y-aviso-de-privacidad.md) | Should | `PA-005`, `PA-021` ✅. **Pendiente: `PA-040`, `PA-045`** |
| `HU-038` | [Administrar usuarios, roles y permisos](EP-005-el-cliente-se-autogestiona/HU-038-administrar-usuarios-roles-y-permisos.md) | Must | `PA-024`, `PA-031` ✅ `actores-y-roles.md` |
| `HU-039` | [Probar la configuración antes de publicar](EP-005-el-cliente-se-autogestiona/HU-039-probar-la-configuracion-antes-de-publicar.md) | Should | `PA-017` ✅ — el simulador entra al alcance |

Orden: `HU-034` → `HU-035` → `HU-039` → `HU-036` → `HU-038` → `HU-037`.

## EP-006 — El expediente vive

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-040` | [Monitoreo continuo](EP-006-el-expediente-vive/HU-040-monitoreo-continuo.md) | Must | `PA-035`, `PA-014`, `PA-011` ✅ `ADR-0009`. **Pendiente: `PA-040`** |
| `HU-041` | [Vencimientos y avisos anticipados](EP-006-el-expediente-vive/HU-041-vencimientos-y-avisos-anticipados.md) | Must | `PA-031` ✅ |
| `HU-042` | [Renovación y actualización periódica](EP-006-el-expediente-vive/HU-042-renovacion-y-actualizacion-periodica.md) | Must | `PA-035`, `PA-025` ✅ `ADR-0009` y `ADR-0004` §2b |
| `HU-043` | [Panel del Oficial de Cumplimiento](EP-006-el-expediente-vive/HU-043-panel-del-oficial-de-cumplimiento.md) | Must | `PA-039` ✅ — KPI estándar + configurables, SLA por etapa |
| `HU-044` | [Exportación del expediente y de reportes](EP-006-el-expediente-vive/HU-044-exportacion-del-expediente-y-de-reportes.md) | Should | `PA-009` ✅ `ADR-0007` · `PA-007` ✅ formatos de exportación |
| `HU-045` | [Recordatorios a la contraparte](EP-006-el-expediente-vive/HU-045-recordatorios-a-la-contraparte.md) | Could | `PA-031`, `PA-028`, `PA-019` ✅ |

Orden: `HU-041` → `HU-040` → `HU-042` → `HU-043` → `HU-044` → `HU-045`.

## EP-007 — Capa comercial

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-046` | [Planes, licencias y cupo de consultas](EP-007-capa-comercial/HU-046-planes-licencias-y-cupo-de-consultas.md) | Must | `PA-015`, `PA-036`, `PA-038` ✅ `ADR-0002` §2b y §2c. **Pendiente: `PA-043`** |
| `HU-047` | [Medición de consumo y control del cupo](EP-007-capa-comercial/HU-047-medicion-de-consumo-y-control-del-cupo.md) | Must | `PA-037`, `PA-036` ✅ — excedente por plan, alertas 80/90/100 % |
| `HU-048` | [Cierre de ciclo y cálculo del excedente](EP-007-capa-comercial/HU-048-cierre-de-ciclo-y-calculo-del-excedente.md) | Must | `PA-036`, `PA-037` ✅ |
| `HU-049` | [Cobro por pasarela](EP-007-capa-comercial/HU-049-cobro-por-pasarela.md) | Must | **`PA-016` ✅ — cambia de fondo:** no hay débito automático. Factura + link de pago (`ADR-0002` §2) |
| `HU-050` | [Factura electrónica](EP-007-capa-comercial/HU-050-factura-electronica.md) | Must | `PA-036`, `PA-038` ✅ — alta manual al inicio |

Orden: el de la lista.

## EP-008 — Salida de datos hacia otros sistemas

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-051` | [Configurar qué campos del expediente son exportables por organización](EP-008-salida-de-datos-hacia-otros-sistemas/HU-051-configurar-campos-exportables-por-organizacion.md) | Must | `PA-010` ✅ origen de la épica. **Pendiente: `PA-048`** (campos bloqueados por política) |
| `HU-052` | [Exportar un expediente cerrado a archivo estructurado (JSON/TXT)](EP-008-salida-de-datos-hacia-otros-sistemas/HU-052-exportar-expediente-a-archivo-estructurado.md) | Must | `PA-010` ✅. **Pendiente: `PA-046`** (medio), `PA-047` (disparador) |
| `HU-053` | [Exponer una interfaz de programación para que el sistema del cliente extraiga expedientes](EP-008-salida-de-datos-hacia-otros-sistemas/HU-053-interfaz-de-programacion-para-extraer-expedientes.md) | Should | `PA-010` ✅. **Pendiente: `PA-046`** (medio) |
| `HU-054` | [Autenticar la integración y dejar bitácora de cada extracción](EP-008-salida-de-datos-hacia-otros-sistemas/HU-054-autenticacion-e-bitacora-de-extracciones.md) | Must | **Pendiente: `PA-046`** (qué medio autenticar) |

Orden sugerido: `HU-051` → `HU-054` → `HU-052` → `HU-053`.

## EP-009 — Presencia pública

| ID | Historia | Prioridad | Dependencia y dónde quedó resuelta |
|----|----------|-----------|---------------|
| `HU-058` | [Página de inicio pública](EP-009-presencia-publica/HU-058-pagina-de-inicio-publica.md) | Should | — (vacío detectado 2026-09-07, no depende de ninguna `PA-xxx`) |
| `HU-059` | [Actualizar la página de inicio con funcionalidad real](EP-009-presencia-publica/HU-059-actualizar-landing-con-funcionalidad-real.md) | Should | Depende de que `EP-003`, `EP-004` o `EP-005` tengan una historia real construida — no depende de ninguna `PA-xxx` |

## Preguntas abiertas que bloqueaban el backlog — estado

Las 39 están **resueltas** (2026-09-05). Estas son las que cambiaron algo del backlog, y qué
cambiaron:

| Pregunta | Historias | Qué quedó decidido |
|---|---|---|
| `PA-016` | `HU-049` | **Cambia de fondo.** No hay cobro desatendido: se factura y se envía link de pago (PSE, tarjeta, Efecty, transferencia). Desaparece la tokenización de tarjetas → `ADR-0002` §2 |
| `PA-025` | `HU-004`, `HU-008`, `HU-034`, `HU-042` | El expediente congela formulario y matriz **al abrirse**, y metodología y reglas **al evaluar**. Lo cerrado es inmutable → `ADR-0004` §2b |
| `PA-026` | `HU-006` | *Append-only* por permisos de base de datos + hash por evento. WORM y hash encadenado quedan como evolución → `ADR-0007` |
| `PA-029` | `HU-009`, `HU-014`, `HU-015`, `HU-019`, `HU-033` | Se puede decidir con pendientes, **nunca en silencio**: excepción con motivo y usuario autorizado. Los requisitos legalmente obligatorios se marcan *hard stop* y no admiten override |
| `PA-033` | `HU-026` | No hay umbral único. Matching multicriterio con tres zonas y confirmación humana → `ADR-0008` |
| `PA-035` | `HU-040`, `HU-042` | Periodicidad por riesgo (alto mensual/trimestral, medio semestral, bajo anual) **más** re-screening por evento → `ADR-0009` |
| `PA-036`, `PA-037` | toda `EP-007` | Modelo híbrido: suscripción + usuarios + créditos + módulos. Excedente facturable en Professional/Enterprise, bloqueo con *upgrade* en Starter, alertas al 80/90/100 % → `ADR-0002` §2b. **Los precios concretos quedan en `PA-043`** |
| `PA-017` | `HU-007`, `HU-029`, `HU-031`, toda `EP-005` | Modelo híbrido: entregamos plantillas base, el cliente ajusta. La interfaz de administración **entra al MVP** → `ADR-0004` §6 |
| `PA-005`, `PA-012` | `EP-003`, `HU-030`, `HU-037`, `HU-040` | Catálogo de fuentes definido; conexión directa preferida; $1.000–$2.000 COP por consulta como cifra **provisional** → `07-integraciones/` |
| `PA-014` | `HU-025`, `HU-040`, `EP-007` | ~1.000 consultas/mes en el primer cliente; capacidad objetivo 10.000–50.000 contrapartes por tenant → `RNF-019` |
| `PA-024` | `HU-001`, `HU-003`, `HU-038` | Seis roles base como plantillas **más** roles propios por permisos granulares; el administrador del tenant los asigna |
| `PA-032` | `HU-017`, `HU-020` | La confianza de la IA es señal, no veredicto. Umbral por campo y tarea; los datos críticos exigen revisión humana → `ADR-0005` §4b |
| `PA-034` | `HU-032`, `HU-036` | Maker-checker **configurable**; obligatorio en riesgo alto, PEP, coincidencia confirmada, DDI y excepciones a controles bloqueantes |
| `PA-039` | `HU-043` | KPI estándar + configurables, con SLA por etapa y semáforos por *aging* |
| `PA-009` | `HU-006`, `HU-013`, `HU-016`, `HU-044` | Retención configurable, referencia 10 años, con copias periódicas al cliente → `ADR-0007`. Decisión formal en `PA-044` |
| `PA-018` | Varias de `EP-001` a `EP-005` | Primer despliegue: 2 estándares (SARLAFT y PTEE) y 7 tipos de contraparte |
| `PA-019`, `PA-020` | `HU-010`, `HU-022`, `HU-045` | Correo + OTP por correo en el MVP; SMS opcional; firma certificada como módulo posterior → `ADR-0010` |
| `PA-021` | `HU-011`, `HU-017`, `HU-018`, `HU-037` | Router de IA por tenant, con opción de desactivarla. El proveedor concreto queda en `PA-045` |
| `PA-027` | `HU-005`, `HU-019`, `HU-021` | Escala de precedencia configurable; frente a la IA manda lo declarado; nunca se borra el origen → `ADR-0005` §2 |
| `PA-028` | `HU-009`, `HU-010`, `HU-014`, `HU-045` | El enlace expirado **no cierra** el expediente: pasa a `Expirado/Pendiente`, con recordatorios y renovación controlada |
| `PA-030` | `HU-013` | PDF, JPG/JPEG, PNG (+ Office si hace falta); 20 MB por archivo configurable; varios archivos por tipo documental → `RNF-023` a `RNF-025` |
| `PA-031` | `HU-008`, `HU-010`, `HU-011`, `HU-015`, `HU-027`, `HU-038`, `HU-041`, `HU-045` | Envío automático por correo **y** copia manual del enlace. Plantillas y marca por tenant |
| `PA-038` | `EP-007` | Alta manual al principio; híbrido después. **El MVP no necesita registro público** |
| `PA-001` | `HU-007` | SARLAFT y SAGRILAFT como núcleo, PTEE como marco complementario, sobre una capa de estándares activables |

### Lo que sigue abierto

| Pregunta | Historias que condiciona | Urgencia |
|---|---|---|
| **`PA-040`** — vía de conexión y costo real de las fuentes colombianas | toda `EP-003`, `HU-030`, `HU-037`, `HU-040`, y por dependencia `PA-043` | **La más urgente.** Bloquea la Fase 3, no la 0 a la 2 |
| **`PA-043`** — precios y cupos de los planes | toda `EP-007` | Antes del bloque comercial. Depende de `PA-040` |
| `PA-041` — firma digital certificada y validez del OTP por correo | `HU-022` | No bloquea: `ADR-0010` arranca con aceptación electrónica |
| `PA-044` — quién asume la retención a 10 años | `HU-006`, `HU-016`, `HU-044` | No bloquea la estructura, sí el contrato |
| `PA-045` — proveedor de IA y su contrato/DPA | `HU-011`, `HU-017`, `HU-018`, `HU-037` | Antes de la Fase 2 |
| **`PA-046`** — medio(s) de salida: interfaz de programación, archivo, o ambos | toda `EP-008` | Bloquea las 4 historias de `EP-008` por igual |
| `PA-047` — disparador de la salida (bajo demanda, programado o por evento) | `HU-052` | No bloquea el disparo manual con el que quedó escrita `HU-052` |
| `PA-048` — campos bloqueados por ser datos personales sensibles o de terceros | `HU-051` | Antes de fijar la lista de campos exportables por defecto |
| `PA-050` — interfaz de confirmación de campos autollenados por extracción | `HU-012`, `HU-017`, `HU-019` | No bloquea nada de Fase 1. Antes de arrancar `EP-002` |

**Las tres de la Fase 0 que más urgían** —`PA-025`, `PA-026` y `PA-029`— están resueltas. La
Fase 0 puede arrancar sin esperar nada.

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
cliente. `EP-005`, `EP-007` y `EP-008` no añaden respuestas a la §44 —añaden autonomía del
cliente, capacidad de cobrar y capacidad de alimentar otros sistemas, respectivamente.
