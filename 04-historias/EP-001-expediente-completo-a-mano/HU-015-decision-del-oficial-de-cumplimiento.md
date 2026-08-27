---
id: HU-015
titulo: Decisión del Oficial de Cumplimiento
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-015 — Decisión del Oficial de Cumplimiento

## Historia

**Como** Oficial de Cumplimiento
**quiero** decidir sobre la vinculación dejando constancia de en qué me basé, con qué
condiciones y hasta cuándo vale
**para** poder sostener esa decisión ante una auditoría años después, con mi nombre y mi cargo
al pie.

## Contexto

La Fase 17 del documento del cliente es tajante: **la decisión siempre es humana y siempre
trazable**. Y la §34 enumera lo que el producto no debe hacer, con dos puntos que aterrizan
aquí: no rechazar automáticamente por aparecer en una lista, y no tratar a la IA como autoridad
de decisión.

`ADR-0005` añade la forma: una decisión **es un evento inmutable**, con persona identificada,
cargo, fundamento, evidencia en la que se basó, condiciones y vigencia. No es una columna
`aprobado` que alguien puede cambiar después. Cambiar de parecer es tomar una decisión nueva,
que se apila sobre la anterior sin borrarla.

Esta historia contesta tres de las dieciséis preguntas del criterio de aceptación del cliente
(§44): **quién decidió, por qué, y con qué condiciones**.

## Criterios de aceptación

```gherkin
Escenario: Aprobar una vinculación
  Dado un expediente en estado "pendiente de decisión"
  Cuando el Oficial de Cumplimiento lo aprueba indicando fundamento, evidencia y vigencia
  Entonces queda registrada la decisión con su nombre, su cargo, la fecha y la hora
  Y el expediente transita a "aprobada"
  Y la decisión queda registrada en la bitácora
```

```gherkin
Escenario: Aprobar con condiciones
  Dado un expediente en estado "pendiente de decisión"
  Cuando el Oficial de Cumplimiento lo aprueba con condiciones
  Entonces la decisión registra cada condición por separado
  Y el expediente transita a "aprobada con condiciones"
  Y las condiciones quedan asociadas a la decisión y son consultables después
```

```gherkin
Escenario: Una decisión sin fundamento no se registra
  Dado un expediente en estado "pendiente de decisión"
  Cuando se intenta decidir sin fundamento o sin indicar en qué evidencia se basó
  Entonces la operación es rechazada indicando qué falta
  Y el expediente no cambia de estado
  Y no queda ninguna decisión a medias
```

```gherkin
Escenario: La decisión es inmutable
  Dado una decisión ya registrada
  Cuando se intenta modificarla o eliminarla
  Entonces la operación es rechazada por la base de datos
  Y la decisión permanece idéntica
```

```gherkin
Escenario: Cambiar de parecer es una decisión nueva
  Dado un expediente con una decisión de aprobación registrada
  Cuando el Oficial de Cumplimiento toma después una decisión de suspensión
  Entonces existen dos decisiones, ambas legibles, en orden
  Y la vigente es la más reciente
  Y la anterior conserva su fundamento, su evidencia y su momento
```

```gherkin
Escenario: Nadie decide sin permiso
  Dado un usuario cuyo rol vigente no incluye el permiso de decidir
  Cuando intenta decidir sobre un expediente
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora con la versión de configuración con la que se evaluó el permiso
```

```gherkin
Escenario: El sistema nunca decide solo
  Dado cualquier expediente en cualquier estado
  Cuando ningún usuario con permiso ha registrado una decisión
  Entonces el expediente no llega por sí mismo a ningún estado de decisión
  Y no existe ningún proceso automático capaz de aprobar, rechazar ni condicionar una vinculación
```

```gherkin
Escenario: Cerrar el expediente
  Dado un expediente con decisión registrada
  Cuando el Oficial de Cumplimiento lo cierra
  Entonces el expediente transita a "cerrada"
  Y su contenido queda accesible para consulta y no admite más escrituras del recorrido
  Y el cierre queda registrado con quién lo hizo y cuándo
```

## Reglas de negocio

- La decisión es **siempre humana** y de una persona identificada, con su cargo en el momento de
  decidir. Ningún proceso automático puede producirla.
- Las opciones son las de la Fase 17: aprobar, aprobar con condiciones, no aprobar, rechazar,
  solicitar más información, suspender, terminar la relación. Cuáles están disponibles en cada
  estado lo declara la máquina de estados (`HU-009`).
- Campos obligatorios de toda decisión: responsable, fecha, **fundamento**, **evidencia en la
  que se basó**, condiciones si aplica, y **vigencia de la vinculación**.
- La decisión es **inmutable**. Una decisión posterior no reemplaza a la anterior: se apila. El
  historial completo es parte del expediente.
- La evidencia citada por la decisión son afirmaciones y documentos concretos del expediente, no
  un texto libre que los mencione.
- La decisión registra la **versión de configuración** con la que se evaluó el expediente. Sin
  eso, la pregunta "¿por qué se le pidió esto?" no tiene respuesta dentro de dos años (§41).
- Un expediente cerrado no admite más escrituras del recorrido. Lo que ocurra después —monitoreo,
  renovación— es Fase 6 y llega por sus propias transiciones.
- La plataforma **no garantiza cumplimiento regulatorio ni emite conceptos jurídicos** (§34,
  §40): registra quién decidió y con qué fundamento.

## Fuera de alcance

- El motor de riesgo que alimenta la decisión con un nivel calculado → Fase 4.
- La debida diligencia intensificada y sus solicitudes adicionales → Fase 4.
- El seguimiento del cumplimiento de las condiciones impuestas → Fase 6.
- La revisión de la decisión cuando vence la vigencia → Fase 6.
- La aprobación por doble control o por comité.
- La notificación de la decisión a la contraparte → depende de `PA-031`.
- La firma electrónica de la contraparte previa a la decisión (Fase 19, nivel 1) → Fase 2. En la
  Fase 1 el expediente se decide sin ella, y conviene decirlo al cliente antes de la
  demostración.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `decision.organization_id` | Sí | Organización cliente existente | No |
| `decision.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `decision.tipo` | Sí | `aprobar` \| `aprobar_con_condiciones` \| `no_aprobar` \| `rechazar` \| `solicitar_mas_informacion` \| `suspender` \| `terminar_relacion` | No |
| `decision.responsable_id` | Sí | Usuario con permiso de decidir | No |
| `decision.cargo` | Sí | Cargo del responsable en el momento de decidir | No |
| `decision.fecha` | Sí | Momento; se escribe una sola vez | No |
| `decision.fundamento` | Sí | Texto no vacío | No |
| `decision.evidencia` | Sí | Al menos una afirmación o documento del expediente | Sí |
| `decision.vigencia_hasta` | Sí | Fecha futura | No |
| `decision.version_configuracion_id` | Sí | Versión con la que se evaluó el expediente | No |
| `condicion.decision_id` | Condicional | Obligatoria si el tipo es `aprobar_con_condiciones` | No |
| `condicion.texto` | Sí | Texto no vacío, una condición por fila | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 17, Fase 15, §30, §34, §40, §44 preguntas L, M y N
- Decisiones: `ADR-0005` (la decisión es un evento inmutable), `ADR-0004` (versión con la que se
  evaluó)
- Supuesto: `SUP-006` (la IA propone y justifica; la decisión la firma una persona)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-029` — bloqueante. Si el Oficial de Cumplimiento puede decidir con
  requisitos pendientes, hace falta registrar esa excepción como parte de la decisión; si no
  puede, hace falta impedirlo en la transición. `PA-031` — si la decisión se notifica y a quién.
  **No pasa de `borrador` hasta que se responda `PA-029`.**
- **Supuestos:** `SUP-006`, `SUP-007` (el posicionamiento es automatizar y trazar la debida
  diligencia, no responder por el cumplimiento).
- **Depende de:** `HU-014` (el expediente está revisado), `HU-009`, `HU-003`, `HU-006`.
- **Habilita a:** `HU-016`, y en la Fase 6 el monitoreo y la renovación.
- **Riesgo:** el atajo tentador es guardar el resultado como una columna del expediente. En el
  momento en que eso ocurre desaparecen el fundamento, la evidencia y el historial, y con ellos
  tres de las dieciséis preguntas de la §44.
