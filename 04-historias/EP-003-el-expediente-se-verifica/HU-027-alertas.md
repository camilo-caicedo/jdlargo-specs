---
id: HU-027
titulo: Alertas
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-027 — Alertas

## Historia

**Como** Analista de Cumplimiento
**quiero** que todo lo que exige mi atención llegue a un mismo sitio, con su origen y su
evidencia
**para** no depender de que alguien se acuerde de mirar cada expediente uno por uno.

## Contexto

La Fase 13 enumera qué genera una alerta: una coincidencia posible o confirmada, una
inconsistencia, un riesgo alto, un documento vencido. La Fase 20 añade los eventos de monitoreo,
que llegan en la Fase 6.

Lo que importa aquí es la distinción de `ADR-0005`, que se pierde con facilidad: **una
coincidencia no es una alerta, y una alerta no es un caso**. El sistema produce coincidencias,
las que superan el criterio configurado se vuelven alertas, y las alertas se agrupan en casos que
una persona resuelve. Colapsar las tres es lo que lleva al rechazo automático que la §34
prohíbe.

Esta historia construye la alerta. El caso es `HU-028`.

## Criterios de aceptación

```gherkin
Escenario: Una coincidencia confirmada genera una alerta
  Dado una coincidencia que el Analista de Cumplimiento confirma
  Cuando se registra la confirmación
  Entonces se genera una alerta asociada al expediente y al sujeto
  Y la alerta cita la coincidencia y su evidencia
  Y queda registrada en la bitácora
```

```gherkin
Escenario: Una discrepancia abierta genera una alerta
  Dado una discrepancia sobre un campo obligatorio que permanece abierta
  Cuando se cumplen las condiciones configuradas para alertar
  Entonces se genera una alerta del tipo inconsistencia
  Y cita las afirmaciones en conflicto
```

```gherkin
Escenario: Un documento vencido genera una alerta
  Dado un documento que transita al estado vencido
  Cuando ocurre la transición
  Entonces se genera una alerta del tipo documento
  Y cita el documento y el requisito que deja de cubrir
```

```gherkin
Escenario: Qué genera alerta es configuración
  Dado dos organizaciones clientes con reglas de alerta distintas configuradas
  Cuando ocurre el mismo hecho en cada una
  Entonces cada una genera las alertas que su configuración define
  Y la diferencia proviene de la configuración, no del programa
```

```gherkin
Escenario: Una alerta no decide nada por sí misma
  Dado una alerta generada de cualquier tipo
  Cuando no ha intervenido ninguna persona
  Entonces el expediente no cambia de decisión
  Y la alerta permanece abierta
  Y no existe ningún proceso automático que la cierre
```

```gherkin
Escenario: Alertas repetidas sobre el mismo hecho no se multiplican
  Dado una alerta abierta sobre un hecho concreto
  Cuando el mismo hecho vuelve a detectarse sin haber cambiado
  Entonces no se genera una alerta duplicada
  Y queda registrado que el hecho se volvió a detectar y cuándo
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las alertas
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta alertas con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Una alerta nace de un hecho concreto y **cita su evidencia**: la coincidencia, la discrepancia,
  el documento o el evento que la originó.
- Qué hechos generan alerta, y con qué criterio, es **configuración** de la organización cliente
  (`ADR-0004`).
- Una alerta **no cambia el estado del expediente ni toma decisiones**. Exige atención; no la
  sustituye.
- **Ningún proceso automático cierra una alerta** (§32). Se cierra dentro de un caso, con
  justificación (`HU-028`).
- Un mismo hecho no genera alertas duplicadas: se registra que se volvió a detectar.
- Toda alerta pertenece a una organización cliente y a un expediente, y hereda su aislamiento.
- Los tipos de alerta previstos son: coincidencia, inconsistencia, documento, riesgo y evento de
  monitoreo. Los dos últimos se llenan en las fases 4 y 6; el tipo existe desde ahora para no
  rehacer la tabla.

## Fuera de alcance

- El análisis, la decisión y el cierre → `HU-028`.
- Las alertas por nivel de riesgo alto → Fase 4, que es cuando se calcula el riesgo.
- Las alertas por evento de monitoreo → Fase 6.
- La notificación de alertas por correo o por otro canal → depende de `PA-031`.
- La priorización automática de alertas y la asignación a analistas.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `alerta.organization_id` | Sí | Organización cliente existente | No |
| `alerta.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `alerta.sujeto_id` | Condicional | Obligatorio si la alerta es sobre una persona concreta | Sí |
| `alerta.tipo` | Sí | `coincidencia` \| `inconsistencia` \| `documento` \| `riesgo` \| `monitoreo` | No |
| `alerta.origen_id` | Sí | Coincidencia, discrepancia, documento o evento que la originó | No |
| `alerta.generada_en` | Sí | Momento; se escribe una sola vez | No |
| `alerta.estado` | Sí | `abierta` \| `en_caso` \| `cerrada` | No |
| `alerta.caso_id` | Condicional | Obligatorio cuando la alerta se agrupa en un caso | No |
| `alerta.redetectada_en` | No | Últimas veces que el mismo hecho se volvió a detectar | No |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: Fase 13, Fase 20, §32, §34, §44 pregunta G
- Decisiones: `ADR-0005` (coincidencia, alerta y caso son tres entidades)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-031` — si las alertas se notifican y a quién. No bloquea el modelo.
  Queda en `borrador` por arrastre de las historias de las que depende.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-026` (coincidencias), `HU-019` (discrepancias), `HU-021` (vencimientos),
  `HU-006`.
- **Habilita a:** `HU-028`, el panel de la Fase 6 y las alertas de monitoreo.
