---
id: HU-059
titulo: Actualizar la página de inicio con funcionalidad real
estado: borrador
epica: EP-009
prioridad: Should
actualizado: 2026-09-07
---

# HU-059 — Actualizar la página de inicio con funcionalidad real

> **Origen.** Se abre al auditar la implementación de `HU-058`: la primera versión de la
> página de inicio incluyó una vista previa del producto ("*showcase*") que simulaba
> screening contra listas restrictivas y una evaluación de riesgo calculada — funcionalidad
> de `EP-003`/`EP-004` que en ese momento **no existía todavía**. Se retiró de `HU-058` por
> ser una promesa de capacidad no construida en un producto de cumplimiento, donde eso es un
> riesgo real, no un detalle de *marketing*. Esta historia es su reemplazo legítimo: la misma
> idea (mostrar el producto en acción), pero solo cuando haya producto real que mostrar.

## Historia

**Como** Visitante evaluando si la plataforma sirve para su caso
**quiero** ver ejemplos representativos de lo que el sistema hace de verdad — expedientes,
verificación, evaluación de riesgo —
**para** juzgar el producto por su funcionalidad real, no por una intención de *roadmap*.

## Contexto

`HU-058` construyó deliberadamente una página mínima: encabezado, propuesta de valor citada de
`vision.md`, tres tarjetas descriptivas y pie de página — sin ninguna captura ni maqueta de
pantalla, porque a esa fecha el producto solo tenía la Fase 1 (`EP-001`) construida y
cualquier maqueta de screening o riesgo habría sido una promesa vacía.

Esta historia se retoma cuando exista contenido real que enseñar: al menos una de
`EP-003` (verificación y *screening*), `EP-004` (relaciones y calificación de riesgo`) o
`EP-005` (autogestión de la configuración) debe estar construida — no hace falta que las tres
lo estén, pero la sección que se agregue debe corresponder exactamente a lo que ya funciona,
nunca a lo que todavía no.

## Criterios de aceptación

```gherkin
Escenario: La vista previa del producto solo muestra capacidades ya construidas
  Dado que se agrega una sección de vista previa a la página de inicio
  Cuando se revisa cada elemento que muestra (screening, riesgo, autogestión, lo que sea)
  Entonces cada uno corresponde a una historia ya implementada y verificada
  Y ninguno describe o simula una capacidad todavía no construida
```

```gherkin
Escenario: Los datos de ejemplo son siempre ficticios
  Dado cualquier dato mostrado en la vista previa (nombres, NIT, resultados)
  Cuando se revisa su origen
  Entonces ningún dato corresponde a una contraparte, cliente u organización real
  Y queda claro por contexto que es un ejemplo, no un caso real
```

```gherkin
Escenario: El contenido nunca promete certificación
  Dado el texto completo de la página de inicio actualizada
  Cuando se revisa contra `ADR-0006`
  Entonces ninguna frase usa "certifica", "certificado" o "certificación" como si el producto
    acreditara o garantizara un resultado
```

## Reglas de negocio

- **Nunca se muestra una capacidad como si funcionara antes de que la historia que la
  construye esté implementada y verificada.** Es la regla que esta historia existe para
  proteger — no se relaja nunca, sea cual sea la presión de tiempo o de venta.
- Los datos de ejemplo son siempre ficticios (mismo criterio que `jdlargo-specs` ya aplica a
  sus propios documentos: nunca nombres, cédulas o NIT reales).
- El diseño visual de la vista previa no reproduce un *mockup* de ventana de navegador
  (barra de URL falsa, puntos de semáforo) — ese patrón quedó explícitamente descartado en la
  auditoría de `HU-058` por genérico. Se resuelve con otro tratamiento visual (capturas
  reales recortadas, ilustración editorial, o simplemente texto + iconografía) que se define
  al construir esta historia, no ahora.

## Fuera de alcance

- **Qué pantallas exactas se muestran.** Depende de qué esté construido cuando se retome esta
  historia — no se decide hoy, se decidiría entonces.
- Cualquier cambio al resto de la página de inicio que `HU-058` ya dejó bien (encabezado,
  propuesta de valor, pie de página) — esta historia solo agrega o reemplaza la sección de
  vista previa.
- Analítica de conversión, A/B testing de la página — no pedido.

## Datos y validaciones

Ninguno propio — depende enteramente de qué datos ya existan en las historias que la
habilitan.

## Trazabilidad

- Épica: `EP-009`
- Capacidad: `CAP-09`
- Decisiones: `ADR-0006`
- Requisito funcional: `RF-059`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio.
- **Depende de:** al menos una de `EP-003`, `EP-004` o `EP-005` con una historia real
  implementada que se pueda mostrar honestamente. **No se puede empezar antes** — es la razón
  de que quede en `borrador`, no `en-revision`.
- **Habilita a:** nada aguas abajo.
- **Riesgo:** el mismo que motivó esta historia — que alguien la construya adelantándose,
  mostrando de nuevo una capacidad que todavía no existe. Cualquier plan de arquitectura para
  esta historia debe verificar contra el estado real del código, no contra el roadmap.
