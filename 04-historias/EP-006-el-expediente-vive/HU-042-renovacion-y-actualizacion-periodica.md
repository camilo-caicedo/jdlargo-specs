---
id: HU-042
titulo: Renovación y actualización periódica
estado: borrador
epica: EP-006
prioridad: Must
actualizado: 2026-08-27
---

# HU-042 — Renovación y actualización periódica

## Historia

**Como** Oficial de Cumplimiento
**quiero** que cada expediente se actualice con la frecuencia que su riesgo merece, y no todos
igual
**para** dedicar el esfuerzo del equipo donde hace falta, en vez de revisar mil contrapartes de
bajo riesgo cada año por costumbre.

## Contexto

Es la Fase 21, y su regla es explícita:

```
Periodicidad = metodología del cliente + estándar aplicable + nivel de riesgo + eventos
```

La §34 lo repite entre las cosas que el MVP no debe hacer: **no asumir una única periodicidad de
actualización para todos**.

Una renovación no es un expediente nuevo desde cero: es volver a recorrer lo que la configuración
exija sobre el mismo expediente, conservando toda su historia. La distinción importa, porque el
§46 recuerda que si un actor relacionado ya está identificado y vigente, no hay que volver a
pedirle todo.

## Criterios de aceptación

```gherkin
Escenario: La periodicidad se calcula, no se fija igual para todos
  Dado dos expedientes de la misma organización cliente con niveles de riesgo distintos
  Cuando se calcula su periodicidad de actualización
  Entonces cada uno obtiene la suya según la metodología, el estándar y su nivel de riesgo
  Y se puede ver de qué regla sale cada plazo
```

```gherkin
Escenario: Llega el plazo de actualización
  Dado un expediente cuya fecha de actualización llega
  Cuando corre el trabajo programado
  Entonces el expediente queda marcado como pendiente de renovación
  Y se genera la alerta correspondiente
  Y la vinculación no se suspende por sí sola
```

```gherkin
Escenario: Un evento extraordinario adelanta la actualización
  Dado un expediente con actualización programada para dentro de meses
  Cuando el monitoreo detecta un cambio que la configuración considera disparador
  Entonces la actualización se adelanta
  Y queda registrado qué evento la adelantó
```

```gherkin
Escenario: Renovar conserva la historia
  Dado un expediente que entra en renovación
  Cuando la contraparte aporta información y documentos nuevos
  Entonces se registran como afirmaciones y versiones nuevas
  Y todo lo anterior se conserva íntegro con su momento
  Y el expediente sigue siendo el mismo, con su identificador original
```

```gherkin
Escenario: La renovación usa la configuración vigente al renovar
  Dado un expediente abierto con la versión 3 de configuración
  Cuando se renueva estando vigente la versión 7
  Entonces la renovación exige lo que define la versión 7
  Y queda registrado que ese tramo del expediente se evaluó con la versión 7
  Y la evaluación original conserva su versión 3
```

```gherkin
Escenario: Una renovación termina con una decisión
  Dado una renovación con su información actualizada y revisada
  Cuando el Oficial de Cumplimiento decide
  Entonces se registra una decisión nueva con su fundamento y su vigencia
  Y la decisión anterior se conserva completa
  Y la vigencia de la vinculación se extiende según la decisión nueva
```

```gherkin
Escenario: No renovar tiene consecuencia registrada, no automática
  Dado un expediente con la renovación vencida y sin respuesta de la contraparte
  Cuando pasa el plazo
  Entonces el expediente queda señalado como desactualizado
  Y se genera la alerta correspondiente
  Y ninguna decisión de suspensión o terminación ocurre sin que una persona la tome
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las renovaciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta renovaciones con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- La periodicidad **se calcula** con la fórmula de la Fase 21: metodología, estándar, nivel de
  riesgo y eventos. Nunca es un plazo único para todos (§34).
- Un evento de monitoreo puede **adelantar** la actualización, y queda registrado cuál lo hizo.
- Renovar **no crea un expediente nuevo**: se trabaja sobre el mismo, conservando su historia y su
  identificador.
- La renovación aplica la **configuración vigente en el momento de renovar**, y ese tramo del
  expediente queda citando esa versión. La evaluación original conserva la suya (`ADR-0004`).
- Una renovación termina con una **decisión nueva** (`HU-015`), con su fundamento y su vigencia.
  La anterior se conserva.
- La falta de renovación **no produce decisiones automáticas**: señala el expediente como
  desactualizado y genera alerta. Suspender o terminar la relación es una decisión humana.
- Lo que ya está vigente y verificado puede reutilizarse dentro de la misma organización cliente
  si la configuración lo permite (§46): no se pide de nuevo lo que sigue siendo válido.

## Fuera de alcance

- La detección de los cambios que disparan la renovación → `HU-040`.
- Los avisos de plazo próximo → `HU-041`.
- Los recordatorios a la contraparte → `HU-045`.
- La suspensión o terminación automática de una vinculación: no existe por diseño.
- La renovación masiva de una cartera completa en un solo acto.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `programacion_actualizacion.organization_id` | Sí | Organización cliente existente | No |
| `programacion_actualizacion.expediente_id` | Sí | Expediente vinculado | No |
| `programacion_actualizacion.proxima_fecha` | Sí | Calculada; nunca fija por defecto para todos | No |
| `programacion_actualizacion.origen_calculo` | Sí | Qué regla produjo el plazo | No |
| `renovacion.expediente_id` | Sí | El mismo expediente, no uno nuevo | No |
| `renovacion.version_configuracion_id` | Sí | Versión vigente al renovar | No |
| `renovacion.disparada_por` | Sí | `calendario` \| `evento` \| `manual` | No |
| `renovacion.estado` | Sí | `pendiente` \| `en_curso` \| `completada` \| `vencida` | No |
| `renovacion.decision_id` | Condicional | Obligatoria al completar | No |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: Fase 21, Fase 20, §34, §46, §44 pregunta O
- Decisiones: `ADR-0004` (periodicidad configurable), `ADR-0005` (la renovación acumula, no
  reemplaza)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-035` (periodicidad y disparadores), `PA-025` (qué versión de
  configuración aplica a un expediente en curso, que aquí reaparece con otra cara). **Queda en
  `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-031` (metodología), `HU-032` (nivel de riesgo), `HU-040` (eventos),
  `HU-041` (plazos), `HU-015` (decisión).
- **Habilita a:** `HU-043` (el panel muestra qué está por vencer y qué desactualizado).
- **Riesgo:** `PA-025` vuelve a aparecer aquí, y con más fuerza: si un expediente abierto congela
  su versión, ¿la renovación la mantiene o adopta la nueva? La respuesta que se dé en `HU-004`
  tiene que servir también para este caso, o habrá dos reglas contradictorias sobre lo mismo.
