---
id: REQ-funcionales
estado: en-revision
actualizado: 2026-09-05
---

# Requisitos funcionales

Qué debe hacer el sistema. Cada `RF-xxx` es verificable y se traza a una capacidad y a las
historias que lo implementan.

> **2026-09-05 — cierre del hueco de trazabilidad.** Este archivo estaba en `TBD` mientras las
> 50 historias ya llevaban meses escritas: la cadena `capacidad → requisito → historia` no
> existía. Los `RF-xxx` de abajo se derivaron leyendo la sección "Historia" y "Reglas de
> negocio" de cada `HU-xxx` — no son requisitos nuevos, son los que ya estaban implícitos en el
> trabajo hecho, ahora numerados y trazados.
>
> **Convención de numeración:** por ahora `RF-0NN` corresponde a `HU-0NN` (una historia, un
> requisito). Es deliberadamente 1:1 para que la trazabilidad sea trivial de verificar. Cuando
> una historia se descomponga en más de un requisito durante la Fase de construcción, el `RF`
> nuevo toma el siguiente número libre de la tabla — nunca se reutiliza ni se renumera (regla
> general de IDs de `CLAUDE.md`).
>
> Prioridad heredada de `04-historias/backlog.md`. Estado `Borrador` en todos: ninguno se marca
> `en-revision` hasta que su historia pase la *Definition of Ready* completa (`jdlargo-specs/CLAUDE.md`).

Prioridad: MoSCoW (Must / Should / Could / Won't).

## EP-000 — Cimientos (`CAP-00`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-001 | Toda persona que actúe en la plataforma está identificada por un correo único y adscrita a al menos una organización cliente mediante una membresía con exactamente un rol; ninguna organización queda sin un miembro Administrador. | `CAP-00` | §30, `ADR-0001` | Must | `HU-001` | Borrador |
| RF-002 | El aislamiento entre organizaciones clientes se impone en la base de datos (RLS) sobre el contexto de usuario propagado en cada transacción, nunca en el código de aplicación; ninguna tabla del dominio admite una consulta sin ese contexto. | `CAP-00` | §31, `ADR-0001` | Must | `HU-002` | Borrador |
| RF-003 | La matriz de permisos por rol es configuración versionada por organización cliente, nunca una lista fija en el programa; toda evaluación de permiso registra la versión de configuración con la que se resolvió. | `CAP-00` | §30, `ADR-0004` | Must | `HU-003` | Borrador |
| RF-004 | La configuración de cumplimiento de cada organización cliente se publica como versiones inmutables (`borrador → publicada → reemplazada`); no existe operación de actualización sobre una versión ya publicada. | `CAP-00` | §41, `ADR-0004` | Must | `HU-004` | Borrador |
| RF-005 | Todo dato del expediente se registra como una afirmación con su origen (`declarado · extraído · verificado · evaluado`), autor, momento y evidencia, en una estructura de solo inserción; ninguna afirmación se sobrescribe. | `CAP-00` | §2, `ADR-0005` | Must | `HU-005` | Borrador |
| RF-006 | Toda acción sobre datos de una organización cliente queda registrada en una bitácora de solo inserción, en la misma transacción que el cambio que describe, incluidos los intentos rechazados por permiso o por aislamiento. | `CAP-00` | §23, `ADR-0007` | Must | `HU-006` | Borrador |

## EP-001 — Un expediente completo, a mano (`CAP-01`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-007 | Los tipos de contraparte y la matriz de requisitos (campo o documento exigido por combinación estándar × tipo) son contenido de una versión de configuración, sin ninguna referencia a un estándar o documento concreto en el código. | `CAP-01` | Fase 2, `ADR-0004` | Must | `HU-007` | Borrador |
| RF-008 | El usuario operativo puede crear una solicitud de vinculación que abre un expediente citando la versión de configuración vigente y derivando sus requisitos de la matriz, sin duplicar sujetos entre organizaciones clientes. | `CAP-01` | §31 | Must | `HU-008` | Borrador |
| RF-009 | El avance del expediente ocurre únicamente mediante transiciones declaradas como datos, cada una persistida con actor, tipo de actor, motivo y versión de configuración vigente; no existe escritura directa sobre el estado. | `CAP-01` | §38 | Must | `HU-009` | Borrador |
| RF-010 | La contraparte accede a su expediente mediante un enlace de un solo uso con token acotado a ese expediente, sin crear cuenta ni contraseña; cada uso queda registrado y solo hay un enlace vigente a la vez. | `CAP-01` | Fase 4 | Must | `HU-010` | Borrador |
| RF-011 | Antes de recibir cualquier dato, la contraparte ve el aviso de privacidad vigente; mostrarlo y que lo acepte son dos hechos distintos, y ambos se registran con versión, fecha, hora y medio. | `CAP-01` | §5, §40 | Must | `HU-011` | Borrador |
| RF-012 | El formulario de identificación se construye dinámicamente desde la matriz de requisitos vigente, guarda cada respuesta como afirmación `declarado`, valida en el servidor y admite guardado parcial mientras el enlace esté vigente. | `CAP-01` | §2, `ADR-0004` | Must | `HU-012` | Borrador |
| RF-013 | La contraparte puede cargar los documentos exigidos por la matriz; cada uno registra huella digital, fechas y emisor declarados, y una carga nueva del mismo tipo documental crea una versión sin sobrescribir la anterior. | `CAP-01` | Fase 7 | Must | `HU-013` | Borrador |
| RF-014 | El Analista puede marcar documentos como válidos, rechazados o pendientes de revisión con motivo obligatorio, y solicitar correcciones como transición del expediente, sin invalidar el resto de lo ya entregado. | `CAP-01` | §30, §46 | Must | `HU-014` | Borrador |
| RF-015 | El Oficial de Cumplimiento registra una decisión sobre la vinculación (aprobar, aprobar con condiciones, no aprobar, rechazar, solicitar más información, suspender o terminar la relación) con responsable, fundamento, evidencia citada y vigencia; la decisión es inmutable y nunca automática. | `CAP-01` | Fase 17 | Must | `HU-015` | Borrador |
| RF-016 | Cualquier expediente puede reconstruirse íntegramente —qué se pidió, qué se entregó, quién revisó, quién decidió y por qué— consultando afirmaciones, transiciones, documentos y bitácora ya existentes, sin fabricar el reporte aparte. | `CAP-01` | Fase 18, §44 | Must | `HU-016` | Borrador |
| RF-055 | Un usuario con cuenta ya creada inicia sesión con correo y contraseña, y si tiene más de una membresía activa elige con cuál organización cliente trabajar antes de ver cualquier dato de dominio; la organización activa siempre es una de sus membresías activas, verificada por el servidor. | `CAP-01` | §30, §31 | Must | `HU-055` | Borrador |
| RF-056 | Un Administrador con permiso `memberships:manage` invita por correo a una persona a su organización cliente con un rol; la membresía solo se activa cuando esa persona acepta explícitamente, nunca antes. | `CAP-01` | §30 | Must | `HU-056` | Borrador |
| RF-057 | Un usuario con cuenta puede solicitar el restablecimiento de su contraseña por correo y fijar una nueva mediante un enlace de un solo uso y vigencia limitada, sin revelar si un correo dado está o no registrado. | `CAP-01` | — | Must | `HU-057` | Borrador |

## EP-002 — El expediente se llena solo (`CAP-02`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-017 | El sistema extrae campos de los documentos cargados y los propone como afirmaciones de origen `extraído`, citando documento fuente y ejecución de IA, sin sobrescribir en ningún caso una afirmación declarada. | `CAP-02` | §32 | Must | `HU-017` | Borrador |
| RF-018 | Toda invocación a un proveedor de IA se registra —éxito o fallo— con proveedor, modelo, versión, plantilla, documento fuente, resultado, confianza y destino de los datos, antes de derivar cualquier afirmación de ella. | `CAP-02` | §32 | Must | `HU-018` | Borrador |
| RF-019 | El sistema muestra lado a lado lo declarado y lo extraído sobre un mismo campo, abre una discrepancia ante cualquier diferencia, y la mantiene abierta hasta que una persona la resuelva con fundamento registrado. | `CAP-02` | §8 | Must | `HU-019` | Borrador |
| RF-020 | Toda afirmación extraída con confianza por debajo del umbral configurado queda pendiente de validación humana; validar o descartar registra quién lo hizo, sin alterar el origen de la afirmación. | `CAP-02` | §32, `ADR-0005` §4b | Must | `HU-020` | Borrador |
| RF-021 | Cada tipo documental tiene una vigencia configurable (duración, fecha propia o sin vencimiento); un documento vencido deja de cubrir su requisito y genera alerta, sin ser borrado. | `CAP-02` | Fase 7 | Must | `HU-021` | Borrador |
| RF-022 | La contraparte puede firmar el formulario completo con nivel 1 (evidencia electrónica con huella e IP) o nivel 2 (más un factor adicional verificado), según la configuración de la organización cliente; una firma no cubre un contenido que cambió después. | `CAP-02` | Fase 19, `ADR-0010` | Should | `HU-022` | Borrador |

## EP-003 — El expediente se verifica (`CAP-03`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-023 | El catálogo de fuentes externas (nombre, proveedor, cobertura, condiciones de uso y costo por consulta) es configuración por organización cliente, con credenciales que nunca llegan al navegador. | `CAP-03` | Fase 11 | Must | `HU-023` | Borrador |
| RF-024 | El sistema contrasta un dato declarado contra una fuente externa y produce una afirmación `verificado` con evidencia congelada (no un enlace), citando la fuente y el costo de la consulta. | `CAP-03` | §34 | Must | `HU-024` | Borrador |
| RF-025 | El sistema consulta a la contraparte contra las listas configuradas por la organización cliente y persiste cada screening como evento inmutable con su respuesta completa, sin rechazar automáticamente por aparecer en una lista. | `CAP-03` | §12.1, §34 | Must | `HU-025` | Borrador |
| RF-026 | Toda coincidencia de nombre o identificador nace en estado `pendiente_revision` y solo pasa a `confirmada` o `descartada` por una persona con fundamento registrado, usando comparación multicriterio (exacta, similitud, alias, transliteración, identificador). | `CAP-03` | §12.3, §12.4, `ADR-0008` | Must | `HU-026` | Borrador |
| RF-027 | Todo hecho que exija atención (coincidencia, discrepancia, documento, riesgo, evento de monitoreo) genera una alerta con su evidencia citada; ninguna alerta se cierra de forma automática. | `CAP-03` | §32 | Must | `HU-027` | Borrador |
| RF-028 | Cada alerta se trabaja dentro de un caso que reúne evidencia, análisis y decisión; ningún caso se cierra sin justificación registrada, y si la configuración lo exige, sin aprobación de un segundo rol. | `CAP-03` | Fase 13, §32 | Must | `HU-028` | Borrador |

## EP-004 — El expediente se califica (`CAP-04`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-029 | El sistema modela las relaciones entre sujetos como entidad de primera clase (tipo, fuente, fecha, porcentaje, evidencia), aplicando a cada tipo de relación el tratamiento que defina el motor de relaciones configurado, sin abrir diligencia automática por cada nombre detectado. | `CAP-04` | Fase 10, `ADR-0004` | Must | `HU-029` | Borrador |
| RF-030 | El sistema sugiere el beneficiario final con su fundamento y evidencia, dejando la determinación final a una persona autorizada; cuando no sea determinable con la evidencia disponible, queda como tal y genera alerta. | `CAP-04` | Fase 9 | Must | `HU-030` | Borrador |
| RF-031 | La metodología de riesgo (factores, ponderaciones, escalas, umbrales, reglas de escalamiento) es configuración versionada por organización cliente, validada por coherencia antes de publicarse. | `CAP-04` | §14, §41, `ADR-0004` | Must | `HU-031` | Borrador |
| RF-032 | El sistema calcula un nivel de riesgo preliminar junto con la explicación de qué reglas se dispararon, dejando la clasificación final a una persona; ningún nivel de riesgo produce por sí solo una decisión sobre la vinculación. | `CAP-04` | §14, §34 | Must | `HU-032` | Borrador |
| RF-033 | El sistema permite activar debida diligencia intensificada por causales configurables, registrando todo lo solicitado y aportado con su procedencia, sin determinar por sí solo la suficiencia de la evidencia. | `CAP-04` | Fase 16, §40 | Must | `HU-033` | Borrador |

## EP-005 — El cliente se autogestiona (`CAP-05`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-034 | El Oficial de Cumplimiento prepara cambios de configuración en un borrador único por organización cliente, compara contra lo vigente en términos de negocio, y solo puede publicar tras pasar la validación de coherencia. | `CAP-05` | `ADR-0004` | Must | `HU-034` | Borrador |
| RF-035 | El Administrador define tipos de contraparte y matriz de requisitos sobre un borrador, viendo el impacto —a qué tipos afecta, cuántos expedientes vigentes— antes de publicar. | `CAP-05` | Fase 6, `ADR-0004` §6 | Must | `HU-035` | Borrador |
| RF-036 | El Oficial de Cumplimiento ajusta factores, pesos y umbrales de su metodología sobre un borrador, con validación de que cada factor tiene fuente de datos y de que los umbrales cubren toda la escala sin huecos. | `CAP-05` | §14 | Must | `HU-036` | Borrador |
| RF-037 | El Administrador activa o desactiva fuentes externas y publica el aviso de privacidad, viendo costo y cobertura antes de activar una fuente. | `CAP-05` | §5, Fase 11 | Should | `HU-037` | Borrador |
| RF-038 | El Administrador invita miembros, les asigna rol y revoca su membresía, sin que nadie pueda otorgar un permiso que no tiene y sin dejar nunca a la organización sin un Administrador. | `CAP-05` | §30 | Must | `HU-038` | Borrador |
| RF-039 | El Oficial de Cumplimiento puede ensayar una configuración en borrador contra casos de prueba, usando el mismo evaluador que la operación real, sin generar expedientes, consultas ni costo reales. | `CAP-05` | `ADR-0004` | Should | `HU-039` | Borrador |

## EP-006 — El expediente vive (`CAP-06`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-040 | El sistema monitorea continuamente las contrapartes que la configuración determine, con la periodicidad y los disparadores configurados, generando alerta y caso ante cualquier cambio detectado, sin decidir por sí solo. | `CAP-06` | Fase 20, `ADR-0009` | Must | `HU-040` | Borrador |
| RF-041 | El sistema avisa con antelación configurable antes de que venza un documento o una vinculación, sin generar avisos duplicados ni consumir consultas externas. | `CAP-06` | Fase 21 | Must | `HU-041` | Borrador |
| RF-042 | La periodicidad de renovación de cada expediente se calcula según metodología, estándar, nivel de riesgo y eventos; renovar trabaja sobre el mismo expediente y termina con una decisión nueva, sin automatizar la suspensión por falta de renovación. | `CAP-06` | Fase 21, §34 | Must | `HU-042` | Borrador |
| RF-043 | El Oficial de Cumplimiento cuenta con un panel de solo lectura, con cifras navegables hasta el detalle, que respeta el aislamiento y los permisos por rol; las métricas concretas quedan definidas en `PA-039`. | `CAP-06` | Fase 22 | Must | `HU-043` | Borrador |
| RF-044 | El Auditor puede exportar el expediente completo y los reportes en un archivo fiel a lo consultable, con su huella digital y registrado en bitácora, respetando permisos y aislamiento. | `CAP-06` | Fase 18 | Should | `HU-044` | Borrador |
| RF-045 | El sistema recuerda automáticamente a la contraparte lo que falta antes de que expire su enlace, con un máximo de recordatorios por expediente y sin repetir lo ya completado. | `CAP-06` | §46 | Could | `HU-045` | Borrador |

## EP-007 — Capa comercial (`CAP-07`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-046 | Cada organización cliente contrata un plan vigente que define cupo de consultas, precio del ciclo y precio del excedente; el cupo aplicable a un consumo es el del plan vigente en el momento de consumir. | `CAP-07` | `ADR-0002` §2b | Must | `HU-046` | Borrador |
| RF-047 | El sistema verifica el cupo disponible antes de ejecutar cualquier consulta que genere costo, y registra el consumo y su evidencia en la misma transacción; el agotamiento del cupo nunca es silencioso. | `CAP-07` | `ADR-0002` §2b, `ADR-0009` | Must | `HU-047` | Borrador |
| RF-048 | El sistema cierra mensualmente el consumo de cada organización cliente, calcula el excedente contra el plan vigente durante ese ciclo, y deja el cierre desglosable consulta por consulta e inmutable una vez cerrado. | `CAP-07` | `ADR-0002` | Must | `HU-048` | Borrador |
| RF-049 | El cobro de cada ciclo se emite como factura con enlace de pago (PSE, tarjeta, Efecty o transferencia), sin exigir registro de tarjeta ni débito automático, con conciliación registrada para los pagos que no cruzan la pasarela. | `CAP-07` | `ADR-0002` §2, `PA-016` | Must | `HU-049` | Borrador |
| RF-050 | Toda factura se emite desde el ciclo cerrado, es inmutable, es rastreable hasta las consultas que la componen, y no se emite si faltan datos de facturación del cliente. | `CAP-07` | `ADR-0002` §3 | Must | `HU-050` | Borrador |

## EP-008 — Salida de datos hacia otros sistemas (`CAP-08`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-051 | Los campos del expediente exportables hacia otro sistema son configuración versionada por organización cliente, con valor por defecto no exportable, y con campos que la plataforma bloquea sin importar la configuración del cliente. | `CAP-08` | `PA-010`, `ADR-0004` | Must | `HU-051` | Borrador |
| RF-052 | El sistema exporta, bajo demanda, un expediente cerrado a un archivo estructurado (JSON o TXT, configurable por organización) que contiene únicamente los campos marcados como exportables. | `CAP-08` | `PA-010` | Must | `HU-052` | Borrador |
| RF-053 | El sistema expone una interfaz de programación de solo lectura para que la organización cliente extraiga sus expedientes cerrados y los campos marcados exportables, con el mismo aislamiento por organización que el resto de la plataforma. | `CAP-08` | `PA-010` | Should | `HU-053` | Borrador |
| RF-054 | Toda extracción de datos hacia otro sistema, por archivo o por interfaz de programación, se autentica con una credencial propia de la organización cliente y queda registrada en la bitácora, incluidos los intentos rechazados. | `CAP-08` | `PA-010`, `ADR-0007` | Must | `HU-054` | Borrador |

## EP-009 — Presencia pública (`CAP-09`)

| ID | Requisito | Capacidad | Origen | Prioridad | Historias | Estado |
|----|-----------|-----------|--------|-----------|-----------|--------|
| RF-058 | Existe una página pública, sin autenticación ni dato de dominio, que explica el producto y enlaza al inicio de sesión, cuyo contenido nunca presenta el producto como certificador. | `CAP-09` | `ADR-0006` | Should | `HU-058` | Borrador |
| RF-059 | La página de inicio pública puede mostrar una vista previa del producto solo cuando corresponde exactamente a funcionalidad ya implementada y verificada; nunca simula una capacidad de una fase todavía no construida. | `CAP-09` | `ADR-0006` | Should | `HU-059` | Borrador |

## Requisitos fuera de esta numeración

- El recorrido exacto de `HU-012` y `HU-017` (si el formulario se autodiligencia desde los
  documentos) depende de `PA-042`. `RF-012` y `RF-017` se redactan sobre el supuesto de que
  ambos caminos son posibles; si `PA-042` cierra a favor de uno solo, se ajustan sin cambiar de
  número.
