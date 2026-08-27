---
id: HU-032
titulo: Evaluación de riesgo y clasificación final
estado: borrador
epica: EP-004
prioridad: Must
actualizado: 2026-08-27
---

# HU-032 — Evaluación de riesgo y clasificación final

## Historia

**Como** Oficial de Cumplimiento
**quiero** ver el nivel de riesgo que calculó el sistema junto con las reglas que se dispararon
para producirlo, y aprobarlo o ajustarlo con fundamento
**para** poder defender esa clasificación ante un supervisor, en lugar de tener que explicar por
qué un programa dijo un número.

## Contexto

La Fase 14 separa dos cosas que suelen confundirse: el **cálculo automático preliminar** y la
**clasificación final**, que queda aprobada por la persona responsable, con posibilidad de
revisar, ajustar de forma justificada, hacer una anulación autorizada o pedir una segunda
aprobación.

Y prohíbe una ruta concreta:

> Nunca debe existir como única ruta: la IA calcula riesgo alto → rechazo automático.

`ADR-0004` añade la regla que hace esto posible y que **no se puede añadir después**: evaluar
nunca devuelve solo un resultado, devuelve **qué reglas se dispararon, con qué datos de entrada y
con qué versión**. Y el evaluador es una **función pura**: recibe configuración y datos, y
devuelve resultado más explicación; no lee ni escribe en la base de datos. Es lo que permite
probarlo con tablas de casos, que es la única forma sensata de confiar en un motor de
cumplimiento.

`ADR-0005` completa: una evaluación es un **evento**. Recalcular no reemplaza la anterior, crea
otra.

## Criterios de aceptación

```gherkin
Escenario: Calcular el riesgo preliminar con su explicación
  Dado un expediente con sus datos y una metodología vigente
  Cuando el sistema evalúa el riesgo
  Entonces produce un nivel preliminar
  Y produce la lista de reglas que se dispararon, con los datos de entrada de cada una
  Y registra la versión de la metodología con la que se evaluó
```

```gherkin
Escenario: El cálculo es preliminar hasta que una persona lo aprueba
  Dado un cálculo de riesgo ya producido
  Cuando ninguna persona lo ha aprobado
  Entonces el expediente muestra el nivel como preliminar
  Y no puede avanzar a decisión apoyándose en él
```

```gherkin
Escenario: Aprobar la clasificación final
  Dado un cálculo preliminar de riesgo
  Cuando el Oficial de Cumplimiento lo aprueba
  Entonces queda registrada la clasificación final con su nombre, su cargo y el momento
  Y el nivel aprobado queda asociado al expediente
  Y el hecho queda registrado en la bitácora
```

```gherkin
Escenario: Ajustar el nivel de forma justificada
  Dado un cálculo preliminar que arroja un nivel
  Cuando el Oficial de Cumplimiento fija un nivel distinto indicando el fundamento
  Entonces queda registrada la anulación justificada con quién la hizo, cuándo, el nivel calculado, el nivel fijado y el motivo
  Y el cálculo original se conserva intacto
  Y la anulación se audita con atención especial
```

```gherkin
Escenario: Nunca hay rechazo automático por riesgo alto
  Dado un expediente cuyo cálculo preliminar arroja el nivel más alto de la escala
  Cuando no ha intervenido ninguna persona
  Entonces el expediente no queda rechazado
  Y se genera una alerta y, si la metodología lo define, se dispara la debida diligencia intensificada
```

```gherkin
Escenario: Recalcular crea una evaluación nueva
  Dado un expediente con una evaluación de riesgo registrada
  Cuando cambian sus datos y se vuelve a evaluar
  Entonces se registra una evaluación nueva con su momento y su versión de metodología
  Y la evaluación anterior se conserva completa
  Y se puede ver qué cambió entre una y otra
```

```gherkin
Escenario: Riesgo inherente y residual
  Dado una metodología que define controles aplicables
  Cuando se evalúa el expediente
  Entonces se obtienen el riesgo inherente, los controles aplicados y el riesgo residual
  Y se identifican los factores críticos que más pesaron
```

```gherkin
Escenario: La explicación permite reconstruir el resultado
  Dado una evaluación registrada hace un año, con una metodología ya reemplazada
  Cuando se consulta
  Entonces se obtienen las reglas que se dispararon, los datos de entrada y la versión aplicada
  Y el resultado se puede reconstruir sin depender de la metodología vigente hoy
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las evaluaciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta evaluaciones de riesgo con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- **Evaluar nunca devuelve solo un número.** Devuelve resultado más explicación: qué reglas se
  dispararon, con qué entradas y con qué versión (`ADR-0004`, regla 5).
- El evaluador es una **función pura**: no lee ni escribe en la base de datos. Se prueba con
  tablas de casos.
- El cálculo es **preliminar**. La **clasificación final** la aprueba una persona con nombre y
  cargo (§14).
- Una anulación justificada registra el nivel calculado, el fijado y el motivo, y **se audita con
  atención especial** (§23).
- Una evaluación es un **evento inmutable**. Recalcular crea otra; no reemplaza.
- El sistema calcula riesgo inherente, controles aplicados, riesgo residual, factores críticos y
  nivel final, según lo que la metodología defina.
- **Ningún nivel de riesgo produce por sí solo una decisión sobre la vinculación** (§34). Puede
  generar alerta y disparar diligencia intensificada; decidir es de `HU-015`.

## Fuera de alcance

- La configuración de la metodología → `HU-031`.
- La debida diligencia intensificada que un nivel alto dispara → `HU-033`.
- El recálculo por cambios posteriores a la vinculación → Fase 6.
- La segunda aprobación como flujo, mientras `PA-034` no defina si se exige y en qué casos.
- La comparación de niveles de riesgo entre clientes o sectores → fase posterior (§36).

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `evaluacion.organization_id` | Sí | Organización cliente existente | No |
| `evaluacion.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `evaluacion.metodologia_version_id` | Sí | Versión con la que se evaluó | No |
| `evaluacion.entradas` | Sí | Datos de entrada usados, congelados | Sí |
| `evaluacion.reglas_disparadas` | Sí | Lista de reglas con su resultado | No |
| `evaluacion.riesgo_inherente` | Sí | Según la escala de la metodología | No |
| `evaluacion.controles_aplicados` | No | Los que la metodología defina | No |
| `evaluacion.riesgo_residual` | Sí | Según la escala | No |
| `evaluacion.factores_criticos` | No | Los que más pesaron | No |
| `evaluacion.ejecutada_en` | Sí | Momento; se escribe una sola vez | No |
| `clasificacion.nivel_final` | Sí | Nivel aprobado | No |
| `clasificacion.aprobada_por` y `cargo` | Sí | Usuario con permiso y su cargo | No |
| `clasificacion.es_anulacion` | Sí | Verdadero si difiere del calculado | No |
| `clasificacion.motivo` | Condicional | **Obligatorio si es anulación** | No |

## Trazabilidad

- Épica: `EP-004`
- Capacidad: `CAP-04`
- Documento del cliente: Fase 14, §23, §34, §42, §44 preguntas I y J
- Decisiones: `ADR-0004` (evaluador puro y explicable), `ADR-0005` (la evaluación es un evento)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-034` — bloqueante para el flujo de aprobación: si la clasificación
  final exige una segunda aprobación y en qué casos. **No pasa de `borrador` hasta que se
  responda.**
- **Supuestos:** `SUP-006`.
- **Depende de:** `HU-031` (metodología), `HU-024` y `HU-026` (datos verificados y coincidencias
  como factores), `HU-029` y `HU-030` (estructura societaria y beneficiario final como factores).
- **Habilita a:** `HU-033`, `HU-015` (la decisión se apoya en el nivel aprobado), y el recálculo
  de la Fase 6.
- **Riesgo:** un motor que devuelve solo el número parece funcionar igual de bien durante meses,
  hasta la primera auditoría. La explicabilidad no se puede añadir después sin reescribir el
  evaluador, y por eso es criterio de aceptación y no una mejora posterior.
