---
id: HU-019
titulo: Conciliación de lo declarado con lo extraído
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-08-27
---

# HU-019 — Conciliación de lo declarado con lo extraído

## Historia

**Como** Analista de Cumplimiento
**quiero** ver lado a lado lo que declaró la contraparte y lo que dice el documento, con la
diferencia señalada
**para** detectar en segundos las inconsistencias que hoy solo aparecen si alguien compara a
mano, papel por papel.

## Contexto

Es la Fase 9 del documento del cliente, y su ejemplo lo explica mejor que cualquier definición:

```
razon_social · "Ficticia S.A.S."           · declarado  · contraparte        · —
razon_social · "Ficticia Logistica S.A.S." · verificado · Cámara de Comercio · doc#0001
→ Inconsistencia — revisión requerida
```

Y su prohibición:

> Nunca debe ocurrir que "la IA corrige automáticamente y el expediente queda limpio" sin que
> quede registro de que hubo una diferencia.

Si `ADR-0005` se respetó, esta historia **no construye nada nuevo**: la conciliación es una
consulta sobre afirmaciones del mismo campo con distinto origen. Eso es exactamente lo que la
decisión prometía a cambio de su costo, y este es el momento en que se cobra.

## Criterios de aceptación

```gherkin
Escenario: Mostrar la comparación lado a lado
  Dado un campo con una afirmación declarada y otra extraída
  Cuando el Analista de Cumplimiento consulta la conciliación del expediente
  Entonces ve para ese campo el valor declarado, el extraído, la diferencia, la fuente y la confianza
  Y ve qué acción se requiere
```

```gherkin
Escenario: Valores coincidentes no generan discrepancia
  Dado un campo cuyas afirmaciones declarada y extraída tienen el mismo valor
  Cuando se calcula la conciliación
  Entonces el campo aparece como concordante
  Y no se abre ninguna discrepancia
  Y ambas afirmaciones siguen existiendo por separado con su origen
```

```gherkin
Escenario: Valores distintos abren una discrepancia
  Dado un campo con una afirmación declarada y otra extraída de valor distinto
  Cuando se calcula la conciliación
  Entonces se abre una discrepancia sobre ese campo
  Y la discrepancia permanece abierta hasta que una persona la resuelva
  Y la apertura queda registrada en la bitácora
```

```gherkin
Escenario: La discrepancia no se cierra sola
  Dado una discrepancia abierta
  Cuando se ejecuta cualquier proceso automático, incluida una nueva extracción
  Entonces la discrepancia sigue abierta
  Y no existe ninguna operación automática que la cierre
```

```gherkin
Escenario: Resolver una discrepancia deja registro y conserva lo descartado
  Dado una discrepancia abierta sobre un campo
  Cuando el Analista de Cumplimiento la resuelve eligiendo una de las afirmaciones y explicando por qué
  Entonces la discrepancia queda cerrada con quién la resolvió, cuándo y con qué fundamento
  Y la afirmación no elegida se conserva como evidencia de que hubo una diferencia
  Y el cierre queda registrado en la bitácora
```

```gherkin
Escenario: Un expediente con discrepancias abiertas no pasa a decisión
  Dado un expediente con al menos una discrepancia abierta sobre un campo obligatorio
  Cuando se intenta llevarlo a "pendiente de decisión"
  Entonces la transición es rechazada indicando qué discrepancias faltan por resolver
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las discrepancias
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta discrepancias con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
  Y un intento de resolver una discrepancia de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- La conciliación es **una consulta**, no una transformación. No escribe valores: lee
  afirmaciones y expone diferencias.
- Toda diferencia entre afirmaciones vigentes del mismo campo abre una **discrepancia**, que
  permanece abierta hasta que una persona la resuelva con fundamento registrado.
- **Ningún proceso automático cierra una discrepancia**, ni la IA, ni una extracción posterior,
  ni una verificación externa. Una verificación puede añadir una afirmación más; resolver sigue
  siendo humano.
- Resolver **no borra** la afirmación descartada: sigue siendo evidencia de que hubo diferencia
  (`ADR-0005`).
- La vista obligatoria es la de la §8: `Declarado | Extraído | Diferencia | Fuente | Confianza |
  Acción requerida`.
- La comparación admite normalización razonable —espacios, mayúsculas, tildes— siempre que la
  normalización quede registrada y no oculte una diferencia real. Ante la duda, se abre la
  discrepancia.
- Un expediente con discrepancias abiertas sobre campos obligatorios no avanza a decisión. Que
  el Oficial de Cumplimiento pueda saltarse esa condición depende de `PA-029`.

## Fuera de alcance

- La regla de precedencia automática entre orígenes → `PA-027`. Mientras no se responda, **toda**
  discrepancia se resuelve a mano, incluso las evidentes.
- La conciliación contra fuentes externas → Fase 3. La estructura ya la admite: es una
  afirmación más, con origen `verificado`.
- La corrección del dato por parte de la contraparte tras ver la discrepancia → forma parte de
  `HU-014` y de la firma previa a la decisión (`HU-022`).
- La detección de inconsistencias entre documentos distintos → fase posterior.
- Las vistas materializadas de "valor vigente" como optimización de consulta.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `discrepancia.organization_id` | Sí | Organización cliente existente | No |
| `discrepancia.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `discrepancia.campo` | Sí | Campo definido en la versión de configuración citada | No |
| `discrepancia.afirmaciones` | Sí | Al menos dos afirmaciones vigentes en conflicto | Sí |
| `discrepancia.estado` | Sí | `abierta` \| `resuelta` | No |
| `discrepancia.abierta_en` | Sí | Momento; se escribe una sola vez | No |
| `resolucion.afirmacion_elegida_id` | Sí | Una de las afirmaciones en conflicto | No |
| `resolucion.fundamento` | Sí | Texto no vacío | No |
| `resolucion.resuelta_por` | Sí | Usuario con permiso de resolver | No |
| `resolucion.resuelta_en` | Sí | Momento; se escribe una sola vez | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: Fase 9, Fase 8 (vista lado a lado), §2, §32
- Decisiones: `ADR-0005` (la conciliación es una consulta, no una función)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-027` — bloqueante para cualquier resolución asistida: cuál es la
  regla de precedencia por defecto y si el cliente puede cambiarla. `PA-029` — si un expediente
  con discrepancias abiertas puede llegar a decisión. **Queda en `borrador`.**
- **Supuestos:** `SUP-006`.
- **Depende de:** `HU-005` (afirmaciones), `HU-012` (lo declarado), `HU-017` (lo extraído).
- **Habilita a:** `HU-020`, y en la Fase 3 la conciliación contra lo verificado.
- **Riesgo:** si esta historia resulta difícil de escribir, la causa no está aquí. Significa que
  `HU-005` guardó las afirmaciones de una forma que no permite compararlas, y el arreglo es allí.
