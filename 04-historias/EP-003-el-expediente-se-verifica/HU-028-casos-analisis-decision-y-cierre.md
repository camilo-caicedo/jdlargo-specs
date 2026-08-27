---
id: HU-028
titulo: Casos — análisis, decisión y cierre
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-028 — Casos: análisis, decisión y cierre

## Historia

**Como** Analista de Cumplimiento
**quiero** trabajar cada alerta dentro de un caso que reúna su evidencia, mi análisis y mi
conclusión
**para** que cerrar una alerta signifique dejar escrito por qué, y no simplemente hacerla
desaparecer de la lista.

## Contexto

La Fase 13 define el caso como lo que agrupa: la alerta, su fuente, la persona o contraparte
afectada, la evidencia, la fecha, quién lo analizó, el análisis, la decisión tomada sobre ese
caso, la justificación, quién aprobó y el cierre.

Y termina con la frase que gobierna esta historia:

> **Nunca se puede cerrar una alerta sin justificación registrada.**

Es la pregunta **H** de la §44 —quién analizó las alertas— y es también donde el producto
demuestra que no automatiza el juicio: el sistema junta la evidencia, la persona la interpreta.

## Criterios de aceptación

```gherkin
Escenario: Abrir un caso a partir de una alerta
  Dado una alerta abierta
  Cuando se abre un caso sobre ella
  Entonces el caso reúne la alerta, su origen, el sujeto afectado y la evidencia asociada
  Y queda registrado quién lo abrió y cuándo
  Y la alerta pasa a estado en caso
```

```gherkin
Escenario: Un caso agrupa varias alertas relacionadas
  Dado dos alertas sobre el mismo sujeto y el mismo hecho
  Cuando se agrupan en un mismo caso
  Entonces el caso las contiene a ambas con su evidencia respectiva
  Y ninguna alerta pertenece a más de un caso abierto a la vez
```

```gherkin
Escenario: Registrar el análisis
  Dado un caso abierto
  Cuando el Analista de Cumplimiento registra su análisis
  Entonces queda guardado con su autor, el momento y la evidencia que consultó
  Y el análisis no reemplaza a ninguno anterior: se apila
```

```gherkin
Escenario: No se puede cerrar un caso sin justificación
  Dado un caso abierto
  Cuando se intenta cerrarlo sin justificación registrada
  Entonces la operación es rechazada
  Y el caso permanece abierto
```

```gherkin
Escenario: Cerrar un caso con justificación
  Dado un caso con su análisis registrado
  Cuando quien tiene el permiso lo cierra indicando la conclusión y la justificación
  Entonces el caso queda cerrado con quién lo cerró, cuándo, la conclusión y la justificación
  Y las alertas que agrupaba quedan cerradas
  Y el cierre queda registrado en la bitácora
```

```gherkin
Escenario: El cierre puede exigir aprobación de otra persona
  Dado una configuración que exige aprobación del Oficial de Cumplimiento para cerrar casos de cierto tipo
  Cuando el Analista de Cumplimiento propone el cierre
  Entonces el caso queda pendiente de aprobación
  Y solo se cierra cuando quien tiene ese permiso lo aprueba
  Y quedan registrados ambos: quien propuso y quien aprobó
```

```gherkin
Escenario: Ningún proceso automático cierra un caso
  Dado un caso abierto de cualquier tipo
  Cuando se ejecuta cualquier proceso automático, incluida una ejecución de IA
  Entonces el caso permanece abierto
  Y no existe ninguna operación automática que lo cierre
```

```gherkin
Escenario: Un caso cerrado se puede reabrir dejando rastro
  Dado un caso cerrado
  Cuando aparece información nueva y quien tiene el permiso lo reabre indicando el motivo
  Entonces el caso queda abierto de nuevo
  Y el cierre anterior se conserva completo con su justificación
  Y la reapertura queda registrada
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los casos
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta casos con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y un intento de cerrar un caso de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Un caso agrupa una o varias alertas, con su fuente, el sujeto afectado, la evidencia, el
  análisis, la decisión, la justificación, quién aprobó y el cierre (Fase 13).
- **Ningún caso se cierra sin justificación registrada.** Es una restricción de la base de datos,
  no una validación de pantalla.
- **Ningún proceso automático cierra un caso ni una alerta** (§32).
- El análisis se apila: registrar uno nuevo no borra el anterior.
- Si la configuración lo exige, el cierre requiere aprobación de otro rol. Quién puede cerrar y
  quién aprobar sale de la matriz de permisos (`HU-003`).
- Reabrir un caso es posible y deja rastro; el cierre anterior no se borra.
- Un caso pertenece a un expediente y a una organización cliente, y hereda su aislamiento.
- La decisión sobre un caso **no es la decisión sobre la vinculación** (`HU-015`). Son dos actos
  distintos, aunque el primero alimente al segundo.

## Fuera de alcance

- La decisión sobre la vinculación → `HU-015`.
- La debida diligencia intensificada que un caso puede disparar → Fase 4.
- Los casos originados en eventos de monitoreo → Fase 6.
- La asignación automática de casos, las colas de trabajo y la medición de tiempos de respuesta →
  Fase 6, con el panel.
- El reporte a autoridad derivado de un caso: no está definido y no se inventa aquí.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `caso.organization_id` | Sí | Organización cliente existente | No |
| `caso.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `caso.sujeto_id` | Sí | Persona o contraparte afectada | Sí |
| `caso.alertas` | Sí | Al menos una alerta; ninguna en dos casos abiertos a la vez | No |
| `caso.abierto_por` y `abierto_en` | Sí | Usuario y momento | No |
| `caso.estado` | Sí | `abierto` \| `pendiente_aprobacion` \| `cerrado` | No |
| `analisis.autor_id` y `registrado_en` | Sí | Usuario y momento; se apila | No |
| `analisis.texto` | Sí | Texto no vacío | Sí |
| `cierre.conclusion` | Sí | Conclusión del caso | No |
| `cierre.justificacion` | Sí | **Texto no vacío. Sin ella no hay cierre** | No |
| `cierre.cerrado_por` | Sí | Usuario con permiso de cerrar | No |
| `cierre.aprobado_por` | Condicional | Obligatorio si la configuración exige aprobación | No |
| `reapertura.motivo` | Sí | Obligatorio al reabrir | No |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: Fase 13, §30, §32, §34, §44 pregunta H
- Decisiones: `ADR-0005` (la alerta no es un caso; las decisiones son eventos inmutables)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna propia. Queda en `borrador` por arrastre de `HU-026` y
  `HU-027`.
- **Supuestos:** `SUP-006` (la IA propone y justifica; la persona decide).
- **Depende de:** `HU-027` (hay alertas), `HU-003` (permisos de cerrar y aprobar), `HU-006`.
- **Habilita a:** la debida diligencia intensificada de la Fase 4 y el panel de la Fase 6.
- **Riesgo:** con volumen alto aparece la presión de cerrar casos en lote con una justificación
  genérica. Eso cumple la letra de la regla y vacía su propósito; conviene que la interfaz no lo
  facilite.
