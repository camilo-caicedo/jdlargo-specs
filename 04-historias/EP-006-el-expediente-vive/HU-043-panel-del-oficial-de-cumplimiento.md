---
id: HU-043
titulo: Panel del Oficial de Cumplimiento
estado: borrador
epica: EP-006
prioridad: Must
actualizado: 2026-08-27
---

# HU-043 — Panel del Oficial de Cumplimiento

## Historia

**Como** Oficial de Cumplimiento
**quiero** una vista que me diga qué exige mi atención hoy y cómo está mi cartera
**para** dirigir al equipo por lo que importa, en lugar de revisar expedientes uno por uno para
enterarme.

## Contexto

Es la Fase 22, y el documento del cliente da las categorías, no las métricas:

- **Expedientes**: total, nuevos, incompletos, en análisis, en diligencia intensificada,
  aprobados, condicionados, rechazados.
- **Riesgo**: distribución por nivel, cambios recientes.
- **Alertas**: por listas, por condición de PEP, por inconsistencias, por documentos, por
  vencimientos, por monitoreo.
- **Gestión**: tiempos de respuesta, pendientes, casos repetidos, productividad del equipo.
- **Auditoría**: decisiones tomadas, modificaciones, anulaciones justificadas, consultas
  realizadas, evidencia disponible.

Qué métrica exacta va en cada categoría y qué tiempos de respuesta se miden es lo que `PA-039`
tiene que responder. Aquí se define la estructura y las reglas, no se inventan los números.

Va al final del roadmap por una razón sensata: un panel sobre datos que todavía no existen es una
pantalla vacía.

## Criterios de aceptación

```gherkin
Escenario: Ver el estado de la cartera
  Dado una organización cliente con expedientes en varios estados
  Cuando el Oficial de Cumplimiento abre su panel
  Entonces ve cuántos expedientes hay en cada estado
  Y puede abrir desde cada cifra la lista de expedientes que la componen
```

```gherkin
Escenario: Ver la distribución de riesgo
  Dado expedientes con clasificación final aprobada
  Cuando se consulta la distribución de riesgo
  Entonces se ve cuántos hay en cada nivel de la escala configurada
  Y se distinguen los que tienen clasificación aprobada de los que solo tienen cálculo preliminar
```

```gherkin
Escenario: Ver las alertas por tipo
  Dado alertas abiertas de distintos tipos
  Cuando se consulta el panel de alertas
  Entonces se ven agrupadas por tipo y por antigüedad
  Y se puede abrir el caso correspondiente desde cada una
```

```gherkin
Escenario: El panel refleja solo lo que el usuario puede ver
  Dado un usuario cuyo rol no le permite ver ciertos expedientes o alertas
  Cuando abre el panel
  Entonces las cifras no incluyen lo que no puede consultar
  Y ninguna cifra revela la existencia de datos a los que no tiene acceso
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el panel
  Dado un Oficial de Cumplimiento miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando abre su panel con su contexto de usuario propagado
  Entonces todas las cifras corresponden únicamente a "Alfa Ficticia S.A.S."
  Y ninguna incluye datos de "Beta Ficticia S.A.S."
```

```gherkin
Escenario: Vista de auditoría
  Dado un periodo determinado
  Cuando el Oficial de Cumplimiento consulta la vista de auditoría
  Entonces ve las decisiones tomadas, las anulaciones justificadas, las modificaciones de configuración y las consultas ejecutadas
  Y puede abrir la evidencia de cada una
```

```gherkin
Escenario: Las cifras se pueden reconstruir
  Dado cualquier cifra del panel
  Cuando se abre su detalle
  Entonces se obtiene la lista exacta de elementos que la componen
  Y esa lista coincide con lo que devuelve la consulta directa sobre los datos
```

```gherkin
Escenario: El panel no altera nada
  Dado cualquier consulta del panel
  Cuando se ejecuta
  Entonces no se modifica ningún expediente, alerta ni caso
  Y la consulta queda registrada en la bitácora
```

## Reglas de negocio

- El panel es **de solo lectura**. No es un lugar desde el que se decida.
- Toda cifra es **navegable**: se puede abrir la lista que la compone. Un número que no se puede
  desglosar no sirve para dirigir a nadie.
- El panel respeta el aislamiento entre organizaciones y los **permisos por rol**: no muestra
  agregados que revelen datos que el usuario no puede ver.
- Las categorías son las cinco de la Fase 22. **Las métricas concretas dependen de `PA-039`** y no
  se inventan aquí.
- Los tiempos de respuesta se miden sobre hechos ya registrados —transiciones, aperturas y cierres
  de caso—, no sobre marcas nuevas creadas para el panel.
- La consulta del panel también deja rastro en la bitácora.

## Fuera de alcance

- La exportación de reportes → `HU-044`.
- Paneles por rol distintos del Oficial de Cumplimiento: analista, auditor, gerencia.
- Alertas y avisos enviados por correo desde el panel → depende de `PA-031`.
- Analítica predictiva, comparativas sectoriales y tendencias → fase posterior (§36).
- La definición de acuerdos de nivel de servicio contractuales: aquí se miden tiempos, no se
  comprometen.

## Datos y validaciones

No introduce entidades nuevas: es una vista sobre lo existente.

| Categoría de la §22 | De dónde salen los datos |
|---|---|
| Expedientes | `HU-008`, `HU-009` (estados y transiciones) |
| Riesgo | `HU-032` (evaluaciones y clasificaciones aprobadas) |
| Alertas | `HU-027`, `HU-028` (alertas y casos) |
| Gestión | `HU-009`, `HU-028` (tiempos entre transiciones y entre apertura y cierre de caso) |
| Auditoría | `HU-006` (bitácora), `HU-015` (decisiones), `HU-032` (anulaciones), `HU-024` y `HU-025` (consultas ejecutadas) |

| Regla | Validación |
|---|---|
| Toda cifra | Debe poder desglosarse en la lista que la compone |
| Agregados | Calculados con el contexto de usuario propagado, nunca con conexión de administrador |
| Métricas concretas | `TBD — PA-039` |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: Fase 22, §30, §23
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (los agregados se leen con contexto de
  usuario, no con conexión de administrador)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-039` — bloqueante: qué indicadores y qué tiempos de respuesta quiere
  ver el Oficial de Cumplimiento. La §22 da categorías, no métricas. **No pasa de `borrador` hasta
  que se responda.**
- **Supuestos:** ninguno propio.
- **Depende de:** prácticamente todas las épicas anteriores. Es una vista sobre lo que ellas
  producen.
- **Habilita a:** `HU-044`.
- **Riesgo:** un panel construido con conexión de administrador "porque los agregados son caros"
  omite las políticas de aislamiento y puede mezclar cifras de dos clientes del SaaS. Es el atajo
  clásico y el fallo más caro posible en este producto.
