---
id: EP-004
titulo: El expediente se califica
estado: borrador
capacidad: CAP-04
actualizado: 2026-08-27
---

# EP-004 — El expediente se califica

## Objetivo

Construir el grafo de relaciones de la contraparte, llegar hasta el beneficiario final, y aplicar
la **metodología de riesgo que cada cliente configura** para obtener un nivel que una persona
aprueba —nunca un veredicto que el sistema impone.

## Por qué existe

Es la Fase 4 del `02-producto/roadmap.md` y la épica donde el producto deja de ser un gestor
documental y se vuelve una herramienta de cumplimiento: hasta aquí el expediente recogía y
contrastaba; ahora **valora**.

Dos ideas del documento del cliente la gobiernan por completo:

1. **No hay una matriz de riesgo universal.** Cada cliente configura factores, ponderaciones,
   escalas, umbrales y reglas de escalamiento (§14, `PA-006` resuelta, `ADR-0004`).
2. **No todo actor relacionado necesita el mismo tratamiento** (Fase 10). Un representante legal
   casi siempre requiere identificación y consulta a listas; un accionista minoritario no
   necesariamente lo mismo que un beneficiario final.

Y una prohibición que atraviesa las cinco historias: *"nunca debe existir como única ruta: la IA
calcula riesgo alto y se rechaza automáticamente"* (§14).

Deja contestadas tres preguntas de la §44: **I** qué metodología se aplicó, **J** qué nivel
resultó, **K** si hubo debida diligencia intensificada.

## Alcance

**Incluye:**

- Personas relacionadas y grafo de relaciones (`HU-029`).
- Identificación del beneficiario final, sujeta a revisión humana (`HU-030`).
- Metodología de riesgo configurable y versionada (`HU-031`).
- Evaluación de riesgo inherente y residual, con su explicación (`HU-032`).
- Debida diligencia intensificada (`HU-033`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Interfaz para que el cliente edite su metodología | Fase 5 | Hasta entonces se carga a mano |
| Re-cálculo del riesgo por cambios posteriores | Fase 6 | Es monitoreo continuo |
| Grafos de relaciones complejos y análisis societario avanzado | Fase posterior (§36) | Fuera del MVP por decisión del cliente |
| Modelos propios de aprendizaje automático para riesgo | Fase posterior (§36) | Explícitamente fuera |
| Análisis transaccional y comportamiento | Fase posterior (§36) | Fuera del MVP |
| Reporte a autoridad derivado de un riesgo alto | Sin definir | No hay definición del cliente; no se inventa |

## Actores involucrados

- **Oficial de Cumplimiento** — configura la metodología, aprueba la clasificación final y activa
  la debida diligencia intensificada.
- **Analista de Cumplimiento** — revisa relaciones, propone y documenta.
- **Sistema** — construye el grafo, sugiere y **calcula un preliminar**. No clasifica.
- **Persona relacionada** — sujeto de la diligencia que el motor de relaciones determine.
- **Auditor / Consulta** — comprueba qué metodología se aplicó y por qué salió ese nivel.

## Criterios de éxito

1. **Cero clasificaciones sin aprobación humana.** El cálculo automático es siempre preliminar.
2. **Cero rechazos automáticos por riesgo alto.**
3. **Toda evaluación es explicable**: qué reglas se dispararon, con qué datos y con qué versión
   de la metodología (`ADR-0004`, regla 5).
4. **Ningún beneficiario final se da por determinado** solo con un documento societario.
5. Dos organizaciones clientes con metodologías distintas obtienen niveles distintos sobre el
   mismo expediente, y la diferencia viene de la configuración.
6. Cada tipo de relación recibe el tratamiento que su configuración define, no el mismo para
   todos.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-029` | Personas relacionadas y grafo de relaciones | Must | borrador |
| `HU-030` | Identificación del beneficiario final | Must | borrador |
| `HU-031` | Metodología de riesgo configurable | Must | borrador |
| `HU-032` | Evaluación de riesgo y clasificación final | Must | borrador |
| `HU-033` | Debida diligencia intensificada | Must | borrador |

Orden sugerido: `HU-031` → `HU-029` → `HU-030` → `HU-032` → `HU-033`. La metodología va primero
porque es la que define qué factores hay que recoger; construir el grafo sin saber qué se pondera
lleva a recoger lo que no se usa.

## Dependencias

- **Épicas:** `EP-000` a `EP-003`.
- **Preguntas abiertas:** `PA-006` (resuelta: cada cliente configura la suya), `PA-034` (si la
  clasificación final exige segunda aprobación), `PA-017` y `PA-018` (quién configura y cuánto),
  `PA-005` (fuentes para el screening de relacionados).
- **Decisiones:** `ADR-0004` (metodología y motor de relaciones son configuración versionada; el
  evaluador es una función pura y explicable), `ADR-0005` (la evaluación es un evento con sus
  entradas y su versión; recalcular no reemplaza, crea otra).
- **Modelo:** `05-datos/modelo-conceptual.md` — las relaciones son aristas de primera clase, con
  consultas recursivas.

## Riesgo abierto

Es la épica donde más fácil resulta construir algo que parece funcionar y no se puede defender.
Un número de riesgo sin la lista de reglas que lo produjeron es indefendible ante un supervisor,
y la explicabilidad **no se puede añadir después**: si el evaluador no la produce desde el primer
cálculo, hay que reescribirlo (`ADR-0004`).
