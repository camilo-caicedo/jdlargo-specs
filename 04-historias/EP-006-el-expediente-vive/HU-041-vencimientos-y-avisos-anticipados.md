---
id: HU-041
titulo: Vencimientos y avisos anticipados
estado: borrador
epica: EP-006
prioridad: Must
actualizado: 2026-08-27
---

# HU-041 — Vencimientos y avisos anticipados

## Historia

**Como** Analista de Cumplimiento
**quiero** que el sistema me avise antes de que un documento o una vinculación venzan
**para** pedir la actualización con tiempo, en vez de enterarme cuando el expediente ya está
incompleto.

## Contexto

`HU-021` construyó las vigencias y el estado `vencido`; esta historia le pone el trabajo
programado que las vigila a diario, tal como lo previó
`08-desarrollo/arquitectura-de-aplicacion.md`: un trabajo diario que revisa vigencias y genera
alertas.

Lo que se vence no es solo un documento. También vence **la vigencia de la vinculación**, que la
decisión fijó en `HU-015`, y el plazo de la próxima actualización periódica (`HU-042`).

Es además la historia más sencilla de toda la Fase 6, y por eso conviene hacerla primero: sirve de
prueba de la infraestructura de trabajos programados que después usan el monitoreo y las
renovaciones.

## Criterios de aceptación

```gherkin
Escenario: Aviso anticipado antes del vencimiento
  Dado un documento con fecha de caducidad y una antelación configurada
  Cuando faltan esos días para que caduque
  Entonces se genera una alerta de vencimiento próximo
  Y la alerta indica qué documento, de qué expediente y para cuándo
```

```gherkin
Escenario: El documento vence
  Dado un documento cuya fecha de caducidad llega
  Cuando corre la revisión diaria
  Entonces el documento transita al estado vencido
  Y se genera la alerta correspondiente
  Y el requisito que cubría aparece como no cubierto
```

```gherkin
Escenario: Vencimiento de la vigencia de la vinculación
  Dado un expediente aprobado con una vigencia definida en su decisión
  Cuando se acerca esa fecha
  Entonces se genera una alerta de actualización requerida
  Y el expediente queda señalado como pendiente de renovación
```

```gherkin
Escenario: La antelación del aviso es configuración
  Dado dos organizaciones clientes con antelaciones distintas configuradas
  Cuando se evalúan los mismos vencimientos
  Entonces cada una recibe sus avisos con su propia antelación
  Y la diferencia no proviene del programa
```

```gherkin
Escenario: No se repiten avisos del mismo vencimiento
  Dado un aviso ya generado sobre un vencimiento
  Cuando la revisión diaria vuelve a ejecutarse sin que nada haya cambiado
  Entonces no se genera un aviso duplicado
  Y queda registrado que la condición sigue vigente
```

```gherkin
Escenario: Renovar un documento cierra su alerta de vencimiento
  Dado una alerta de documento vencido
  Cuando la contraparte carga una versión vigente de ese tipo documental y se valida
  Entonces la alerta se puede cerrar dentro de su caso, con justificación
  Y el requisito vuelve a aparecer como cubierto
```

```gherkin
Escenario: El trabajo diario es reintentable
  Dado una ejecución del trabajo de vencimientos que falla a mitad
  Cuando se reintenta
  Entonces no se duplican alertas ya generadas
  Y se procesan los expedientes que quedaron pendientes
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los vencimientos
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta alertas de vencimiento con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Un trabajo programado diario revisa vigencias de documentos, vigencias de vinculación y plazos
  de actualización.
- La **antelación** del aviso es configuración de la organización cliente, por tipo documental si
  hace falta. No todos los documentos necesitan el mismo margen.
- Un vencimiento genera **alerta**, nunca una decisión ni un cambio de estado de la vinculación.
- No se generan avisos duplicados sobre la misma condición: se registra que persiste.
- El trabajo es **idempotente y reintentable** (`ADR-0001`): un reintento no duplica alertas.
- Cerrar una alerta de vencimiento ocurre dentro de su caso, con justificación (`HU-028`).
- Los vencimientos no consumen consultas externas: son cálculo interno y **no generan costo
  variable**. Es la diferencia con `HU-040`.

## Fuera de alcance

- El monitoreo que consulta fuentes externas → `HU-040`.
- La renovación del expediente como proceso → `HU-042`.
- Los recordatorios a la contraparte → `HU-045`.
- La renovación automática de un documento sin intervención: no existe, siempre la aporta alguien.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `vencimiento.organization_id` | Sí | Organización cliente existente | No |
| `vencimiento.objeto` | Sí | `documento` \| `vinculacion` \| `actualizacion` | No |
| `vencimiento.objeto_id` | Sí | Documento, expediente o programación afectada | No |
| `vencimiento.fecha` | Sí | Fecha de caducidad o de plazo | No |
| `vencimiento.antelacion_dias` | Sí | Configurable por organización cliente y tipo | No |
| `vencimiento.estado` | Sí | `proximo` \| `vencido` \| `resuelto` | No |
| `vencimiento.alerta_id` | Sí | Alerta generada | No |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: Fase 7, Fase 20, Fase 21, §34, §44 pregunta O
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (trabajo diario de vencimientos),
  `ADR-0004` (la antelación es configuración)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-031` — si los avisos se notifican por correo y a quién. No bloquea
  el modelo. Queda en `borrador` por arrastre de las historias de las que depende.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-021` (vigencias), `HU-015` (vigencia de la vinculación), `HU-027`
  (alertas).
- **Habilita a:** `HU-042`, `HU-043`, `HU-045`.
