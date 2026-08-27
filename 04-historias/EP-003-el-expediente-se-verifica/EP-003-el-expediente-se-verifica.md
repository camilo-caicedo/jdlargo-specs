---
id: EP-003
titulo: El expediente se verifica
estado: borrador
capacidad: CAP-03
actualizado: 2026-08-27
---

# EP-003 — El expediente se verifica

## Objetivo

Contrastar lo que dice la contraparte contra fuentes externas autorizadas, consultar las listas
que cada cliente configure, y convertir cada coincidencia que exija análisis en un **caso** que
una persona resuelve y cierra con justificación.

## Por qué existe

Es la Fase 3 del `02-producto/roadmap.md` y donde aparece el tercer origen del dato: **lo
verificado**. Hasta aquí el expediente solo contenía lo que la contraparte declaró y lo que la
IA leyó; ninguna de las dos cosas es confirmación de nada.

Es también la épica con más superficie de error de interpretación, y el documento del cliente
dedica media §34 a acotarla: **no rechazar automáticamente por aparecer en una lista**, no
declarar positivos automáticos, no depender de una sola fuente para verificar identidad, y no
limitar el screening a dos listas internacionales.

La cadena correcta, que esta épica implementa literalmente:

```
Coincidencia técnica → revisión humana → comparación de identificadores
→ descartada | probable | confirmada → decisión
```

Deja contestadas cuatro preguntas de la §44: **E** qué se verificó, **F** qué fuentes se
consultaron, **G** qué alertas aparecieron, **H** quién las analizó.

## Alcance

**Incluye:**

- Catálogo de fuentes configurable, con costo y condiciones de uso (`HU-023`).
- Verificación de datos declarados contra esas fuentes, con evidencia congelada (`HU-024`).
- Screening contra listas, PEP y sanciones (`HU-025`).
- Comparación de nombres e identificadores, con sus cinco estados (`HU-026`).
- Alertas (`HU-027`).
- Casos: análisis, decisión y cierre con justificación (`HU-028`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Cálculo de riesgo a partir de lo verificado | Fase 4 | Verificar no es calificar |
| Personas relacionadas y beneficiario final | Fase 4 | Screening de relacionados exige antes el grafo |
| Monitoreo continuo y re-consulta periódica | Fase 6 | Aquí la consulta es puntual; la vigilancia es después |
| Cuotas, cobro y control de cupo de consultas | Fase C | Aquí se registra el costo de cada consulta; cobrarlo es otra épica |
| Análisis avanzado de medios adversos | Fase posterior (§36) | Fuera del MVP por decisión del cliente |
| Auto-hospedar el motor de listas | Sin fase | `ADR-0001` lo descarta explícitamente en esta etapa |

## Actores involucrados

- **Administrador** — configura el catálogo de fuentes.
- **Sistema** — consulta, compara y produce coincidencias. **Nunca confirma una coincidencia.**
- **Analista de Cumplimiento** — analiza los casos y decide sobre cada coincidencia.
- **Oficial de Cumplimiento** — resuelve lo que se le escala y aprueba los cierres que la
  política le reserve.
- **Auditor / Consulta** — comprueba qué se consultó, cuándo y con qué respuesta.

## Criterios de éxito

1. **Ninguna coincidencia se confirma sola.** No existe camino por el que el sistema marque a
   alguien como sancionado sin que una persona lo haya comparado y decidido.
2. **Ningún rechazo automático por lista.** Aparecer en una lista genera un caso, nunca una
   decisión.
3. **Toda consulta externa deja evidencia congelada**: la respuesta tal como llegó, no un enlace
   a la respuesta.
4. **Ningún caso se cierra sin justificación registrada.**
5. Las fuentes son datos: añadir una fuente nueva no exige desplegar código.
6. Un dato verificado se distingue siempre de uno declarado, de uno extraído y de uno **no
   verificable**.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-023` | Catálogo de fuentes externas | Must | borrador |
| `HU-024` | Verificación de datos contra fuentes externas | Must | borrador |
| `HU-025` | Screening contra listas, PEP y sanciones | Must | borrador |
| `HU-026` | Comparación de nombres e identificadores | Must | borrador |
| `HU-027` | Alertas | Must | borrador |
| `HU-028` | Casos: análisis, decisión y cierre | Must | borrador |

Orden sugerido: `HU-023` → `HU-027` → `HU-028` → `HU-024` → `HU-025` → `HU-026`. Las alertas y
los casos se adelantan porque son el destino de todo lo demás: sin ellos, una coincidencia no
tiene dónde caer.

## Dependencias

- **Épicas:** `EP-000`, `EP-001`, `EP-002`.
- **Preguntas abiertas:** `PA-005` (qué fuentes y con qué proveedor), `PA-012` (cuánto cobra un
  proveedor local colombiano), `PA-033` (umbral de similitud del matching), `PA-014` (cuántas
  contrapartes maneja un cliente típico, que determina el costo real).
- **Decisiones:** `ADR-0001` (proveedor internacional consolidado; no auto-hospedar; cada
  screening se persiste como evento inmutable con su costo), `ADR-0005` (lo verificado es un
  origen, la coincidencia no es una alerta y la alerta no es un caso).
- **Integraciones:** `07-integraciones/README.md`, donde vive el riesgo de costo.

## Riesgo abierto

Es la épica que introduce **costo variable por uso**. La §31 prohíbe reutilizar datos de
contrapartes entre clientes del SaaS, así que la deduplicación de consultas solo puede darse
dentro de una misma organización cliente: si dos clientes verifican al mismo proveedor, se paga
dos veces, sin excepción. Con `PA-005`, `PA-012` y `PA-014` abiertas, **el costo por cliente no
se puede estimar todavía**, y de eso depende que el modelo comercial de `EP-007` cierre.
