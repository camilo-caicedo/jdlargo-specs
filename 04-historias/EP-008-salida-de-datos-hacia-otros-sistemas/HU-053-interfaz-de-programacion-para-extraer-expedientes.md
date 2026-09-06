---
id: HU-053
titulo: Exponer una interfaz de programación para que el sistema del cliente extraiga expedientes
estado: borrador
epica: EP-008
prioridad: Should
actualizado: 2026-09-06
---

# HU-053 — Exponer una interfaz de programación para que el sistema del cliente extraiga expedientes

## Historia

**Como** Administrador de una organización cliente
**quiero** que el otro sistema de mi organización pueda consultar, mediante una interfaz de
programación, los expedientes cerrados y sus campos exportables
**para** integrar la información sin depender de que alguien descargue y suba archivos a mano.

## Contexto

`PA-010` plantea la interfaz de programación como la otra vía posible de salida, junto al archivo
estructurado (`HU-052`). Las dos comparten las mismas reglas de fondo — qué expediente puede
salir y qué campos —, definidas en `HU-051`; esta historia solo cambia el medio de entrega.

El cliente mismo marca esta vía como evaluable, no como decidida: "no necesariamente tiene que
ser mediante API". Por eso queda en prioridad **Should** y no **Must**: el archivo estructurado
(`HU-052`) cubre el caso mínimo; esta historia es la vía más costosa de construir y solo se
justifica si `PA-046` confirma que el cliente la necesita.

## Criterios de aceptación

```gherkin
Escenario: Consultar un expediente cerrado por la interfaz de programación
  Dado un expediente cerrado con campos marcados como exportables
  Cuando el sistema del cliente lo consulta con una credencial válida de su organización
  Entonces recibe los campos marcados como exportables de ese expediente
  Y no recibe ningún campo que no esté marcado como exportable
```

```gherkin
Escenario: Rechazar la consulta de un expediente de otra organización
  Dado una credencial de integración de "Alfa Ficticia S.A.S."
  Cuando se usa para consultar un expediente de "Beta Ficticia S.A.S."
  Entonces la consulta se rechaza
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Rechazar la consulta de un expediente que no está cerrado
  Dado un expediente en cualquier estado distinto de cerrado
  Cuando se consulta por la interfaz de programación
  Entonces la interfaz responde que el expediente no está disponible para salida
```

```gherkin
Escenario: La interfaz es de solo lectura
  Dado una credencial de integración válida
  Cuando se intenta usarla para modificar un expediente
  Entonces la operación se rechaza
  Y la interfaz nunca acepta escritura sobre el expediente
```

## Reglas de negocio

- La interfaz de programación entrega **exactamente** lo que entregaría el archivo estructurado:
  mismos expedientes elegibles (cerrados) y mismos campos (marcados exportables en `HU-051`). No
  hay dos conjuntos de reglas de exportación, uno por medio.
- El acceso es **de solo lectura**: nunca permite crear, modificar ni cerrar un expediente.
- El aislamiento por organización aplica igual que en el resto de la plataforma (RLS): una
  credencial nunca alcanza datos de otra organización.
- Todo acceso, exitoso o rechazado, queda registrado en la bitácora (`HU-054`).

## Fuera de alcance

- La autenticación y el ciclo de vida de las credenciales de integración → `HU-054`.
- Notificación por evento (webhook) cuando cambia un expediente — depende de `PA-047`; esta
  historia es de consulta (`pull`), no de aviso (`push`).
- Límites de tasa, cuotas de la interfaz y acuerdos de nivel de servicio detallados: `TBD`.
- Cualquier operación de escritura.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `consulta_integracion.organization_id` | Sí | Debe coincidir con la organización de la credencial usada | No |
| `consulta_integracion.expediente_id` | Sí | Expediente en estado cerrado | No |
| `consulta_integracion.campos_solicitados` | Sí | Subconjunto de los campos marcados exportables (`HU-051`) | Depende del campo |
| `consulta_integracion.resultado` | Sí | `entregado` \| `rechazado`, con motivo | No |

## Trazabilidad

- Épica: `EP-008`
- Capacidad: `CAP-08`
- Preguntas: implementa la segunda de las dos vías propuestas en `PA-010`

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-046`** — sin confirmar que el cliente prefiere (o necesita) esta
  vía, construirla es la apuesta más cara de las dos posibles.
- **Depende de:** `HU-051` (campos exportables), `HU-054` (autenticación y bitácora).
- **Riesgo:** es la historia más costosa de esta épica; si `PA-046` cierra a favor de solo el
  archivo estructurado, esta historia se puede posponer sin bloquear el resto de `EP-008`.
