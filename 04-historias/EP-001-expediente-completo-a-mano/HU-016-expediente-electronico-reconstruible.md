---
id: HU-016
titulo: Expediente electrónico reconstruible
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-016 — Expediente electrónico reconstruible

## Historia

**Como** Auditor
**quiero** abrir un expediente y ver toda su historia en orden, sin pedirle nada a nadie
**para** comprobar por mí mismo qué se pidió, qué entregó la contraparte, quién lo revisó, quién
decidió y por qué.

## Contexto

Es la Fase 18 del documento del cliente, y su exigencia se lee mejor tal cual la escribió:

> Qué se preguntó → qué respondió la contraparte → qué documento entregó → qué verificó el
> sistema → qué alerta apareció → quién la analizó → qué decisión se tomó → por qué → qué pasó
> después.

En la Fase 1 ese recorrido está recortado: no hay verificación ni alertas todavía. Lo que sí
tiene que estar completo es el tramo que sí existe, y **tiene que salir del propio expediente**,
no de un reporte fabricado aparte. Esa es la diferencia entre el criterio de "cumplimiento
verificable" de la §42 y un `Cumplido: Sí/No`.

La buena noticia es que si `EP-000` se construyó bien, esta historia es sobre todo una consulta:
las afirmaciones ya tienen su origen, las transiciones ya están registradas y la bitácora ya
existe. Si hubiera que fabricar datos aquí, sería la señal de que algo se guardó mal antes.

## Criterios de aceptación

```gherkin
Escenario: Reconstruir el recorrido completo de un expediente
  Dado un expediente cerrado de "Alfa Ficticia S.A.S."
  Cuando el Auditor lo consulta
  Entonces ve qué requisitos exigía la versión de configuración citada
  Y qué declaró la contraparte, con el origen y el momento de cada afirmación
  Y qué documentos entregó, con su huella digital y su estado
  Y quién lo revisó y qué observó
  Y qué decisión se tomó, por quién, con qué fundamento, qué condiciones y hasta cuándo vale
  Y todo aparece en orden cronológico, sin huecos desde la apertura
```

```gherkin
Escenario: Las seis preguntas de la Fase 1 se contestan desde el expediente
  Dado un expediente cerrado
  Cuando se consulta
  Entonces se puede responder quién era la contraparte
  Y qué información entregó
  Y qué documentos presentó
  Y quién tomó la decisión
  Y por qué la tomó
  Y qué condiciones quedaron
  Y ninguna de esas respuestas exige construir un reporte aparte
```

```gherkin
Escenario: Cada dato muestra su procedencia
  Dado un expediente con afirmaciones registradas
  Cuando se consulta cualquier dato de la contraparte
  Entonces se muestra si fue declarado, extraído, verificado o evaluado
  Y quién o qué lo produjo y cuándo
  Y un dato declarado nunca se presenta como verificado
```

```gherkin
Escenario: La configuración con la que se armó sigue siendo legible
  Dado un expediente que citó la versión 3 de configuración
  Y que la organización cliente ya publicó la versión 7
  Cuando se consulta el expediente
  Entonces se muestran los requisitos tal como los definía la versión 3
  Y se indica explícitamente con qué versión fue armado y evaluado
```

```gherkin
Escenario: El Auditor consulta y no toca
  Dado un usuario con rol de Auditor
  Cuando consulta un expediente cerrado y su bitácora
  Entonces obtiene el contenido completo de la organización cliente
  Y cualquier intento suyo de modificar el expediente es rechazado
  Y su consulta queda registrada en la bitácora
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la consulta del expediente
  Dado un auditor miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta expedientes con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y no obtiene ningún expediente de "Beta Ficticia S.A.S."
```

```gherkin
Escenario: Un documento consultado es el mismo que se entregó
  Dado un documento entregado en un expediente cerrado
  Cuando el Auditor lo descarga
  Entonces obtiene el archivo original tal como se cargó
  Y su huella digital coincide con la registrada en el momento de la carga
```

## Reglas de negocio

- El expediente **conserva la información original**, los documentos originales con sus
  versiones, las decisiones, los registros con fecha y hora, quiénes intervinieron, los cambios
  realizados y la evidencia del consentimiento (Fase 18).
- La reconstrucción **se consulta, no se fabrica**: sale de las afirmaciones, las transiciones,
  los documentos y la bitácora que ya existen. Si algún dato hubiera que calcularlo desde cero
  aquí, es que se guardó mal antes.
- Cada dato se muestra con su origen. **Lo declarado no se presenta como verificado** (§2). Es
  una regla de pantalla, además de una regla de modelo.
- El expediente indica siempre con qué versión de configuración fue armado y evaluado, y esa
  versión sigue siendo legible aunque haya sido reemplazada (§41).
- El Auditor tiene lectura completa, incluida la bitácora, y ninguna capacidad de escritura
  (§30).
- La consulta de un expediente también deja rastro. Quién miró qué es parte de la auditoría.
- El expediente debe poder conservarse de forma íntegra y accesible en el tiempo. La normativa
  colombiana sobre mensajes de datos aplica aquí `(por validar — Ley 527 de 1999, ver SUP-008)`.

## Fuera de alcance

- La exportación del expediente a PDF o Excel → Fase 6, con el panel del Oficial de
  Cumplimiento. Aquí se reconstruye en pantalla.
- El panel con indicadores, colas y estadísticas → Fase 6.
- El portal para auditores externos, que el propio cliente ubica en una fase posterior (§36).
- El tramo del recorrido que todavía no existe: verificaciones, fuentes consultadas, alertas y
  su análisis, metodología de riesgo, nivel resultante y debida diligencia intensificada. Son
  las preguntas D a K de la §44 y llegan en las fases 2 a 4.
- La política de retención del expediente y de sus archivos → depende de `PA-009`.

## Datos y validaciones

No introduce entidades nuevas. Consulta las de las historias anteriores:

| Origen del dato | Historia | Qué aporta a la reconstrucción |
|---|---|---|
| Requisitos de la matriz | `HU-007` | Qué se preguntó y por qué |
| Expediente y sujeto | `HU-008` | Quién era la contraparte (§44 A) |
| Transiciones | `HU-009` | Cuándo avanzó y por quién |
| Usos del enlace de acceso | `HU-010` | Cuándo entró la contraparte y desde dónde |
| Evidencia del consentimiento | `HU-011` | Qué versión del aviso aceptó y cuándo |
| Afirmaciones declaradas | `HU-012` | Qué información entregó (§44 B) |
| Documentos y sus huellas | `HU-013` | Qué documentos presentó (§44 C) |
| Revisiones y observaciones | `HU-014` | Quién revisó y qué observó |
| Decisión y condiciones | `HU-015` | Quién decidió, por qué y con qué condiciones (§44 L, M, N) |
| Bitácora | `HU-006` | El hilo que une todo lo anterior |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 18, §42, §44 preguntas A, B, C, L, M y N, §30
- Decisiones: `ADR-0005` (las 16 preguntas se responden por construcción)
- Normativa: conservación de mensajes de datos `(por validar — Ley 527 de 1999, ver SUP-008)`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-009` — retención y trazabilidad: cuánto tiempo debe conservarse un
  expediente cerrado y en qué condiciones. No bloquea la consulta, sí la política de
  conservación. **Queda en `borrador` mientras siga abierta y mientras lo estén las historias de
  las que depende.**
- **Supuestos:** `SUP-008` (las referencias normativas no están verificadas).
- **Depende de:** todas las historias anteriores de `EP-001`, y de `HU-006`.
- **Habilita a:** la demostración con la que arranca la Fase 2, que según el roadmap se hace con
  un caso real y no con datos de prueba.
- **Riesgo:** es la historia que revela si las demás se hicieron bien. Si aquí hace falta
  inventar una consulta rara para responder una de las seis preguntas, el problema no está aquí
  sino en la historia que guardó ese dato.
