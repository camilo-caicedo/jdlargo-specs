---
id: EP-002
titulo: El expediente se llena solo
estado: borrador
capacidad: CAP-02
actualizado: 2026-08-27
---

# EP-002 — El expediente se llena solo

## Objetivo

Que la IA lea los documentos que entrega la contraparte y proponga los datos, que el sistema
**haga visible cada diferencia** entre lo que se declaró y lo que dice el documento, y que
ninguna de las dos cosas se resuelva sin que una persona lo confirme.

## Por qué existe

Es la Fase 2 del `02-producto/roadmap.md` y el primer punto donde la plataforma ahorra trabajo
de verdad: dejar de transcribir a mano lo que ya está escrito en una cédula o en un certificado.

También es donde el producto se puede romper de la peor manera posible, y el documento del
cliente lo dice con una regla de oro:

> La IA **nunca sobrescribe en silencio** lo que la contraparte declaró a mano.

Nada de esto es viable si `ADR-0005` no se cumplió antes: la conciliación no es una función que
haya que escribir, es una consulta sobre afirmaciones del mismo campo con distinto origen. Si
`EP-000` guardó valores planos, esta épica no se puede construir.

Deja contestada la pregunta **D** de la §44: qué información fue extraída automáticamente.

## Alcance

**Incluye:**

- Extracción de campos desde los documentos cargados (`HU-017`).
- Registro completo de cada ejecución de IA, según la §32 (`HU-018`).
- Conciliación de lo declarado con lo extraído, y discrepancias visibles (`HU-019`).
- Validación humana de lo extraído antes de que valga como dato del expediente (`HU-020`).
- Vigencias y ciclo completo de estados del documento, incluido el vencido (`HU-021`).
- Firma electrónica de niveles 1 y 2 sobre el expediente (`HU-022`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Verificación contra fuentes externas | Fase 3 | Extraer no es verificar: son dos orígenes distintos (§2) |
| Firma digital certificada de nivel 3 | Sin fase asignada | Es una integración con un tercero acreditado, con costo por firma → `PA-020` |
| Extracción de estructuras societarias y relaciones | Fase 4 | El grafo de relaciones necesita su propio modelo |
| Modelos de IA propios o entrenados | Fase posterior (§36) | El MVP usa modelos de terceros tras un puerto intercambiable |
| Cálculo de riesgo a partir de lo extraído | Fase 4 | — |
| Comparación de documentos entre sí y detección de fraude documental | Fase posterior | La §8 la menciona como capacidad; no está en el alcance del MVP |

## Actores involucrados

- **Sistema** — ejecuta la extracción y calcula las discrepancias. Nunca confirma nada.
- **Analista de Cumplimiento** — valida o descarta lo que propuso la IA.
- **Contraparte** — revisa el formulario ya autollenado, corrige y firma.
- **Oficial de Cumplimiento** — decide sobre las discrepancias que el analista escala.
- **Auditor / Consulta** — puede ver qué modelo leyó qué documento y con qué confianza.

## Criterios de éxito

1. **Cero sobrescrituras silenciosas.** No existe ninguna operación en la que un dato extraído
   reemplace a uno declarado sin dejar ambas afirmaciones visibles.
2. **Cero datos de IA sin trazabilidad.** Toda afirmación de origen `extraído` cita su ejecución
   de IA, con modelo, proveedor, versión, documento fuente y confianza.
3. **Toda discrepancia es visible y permanece abierta** hasta que una persona la resuelva con
   registro.
4. **Ningún dato extraído asciende a verificado** sin una acción humana o una fuente externa.
5. La vista `Declarado | Extraído | Diferencia | Fuente | Confianza | Acción requerida` existe
   para cualquier campo conciliable, tal como la pide la §8.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-017` | Extracción de datos desde los documentos | Must | borrador |
| `HU-018` | Registro de cada ejecución de IA | Must | borrador |
| `HU-019` | Conciliación de lo declarado con lo extraído | Must | borrador |
| `HU-020` | Validación humana de lo extraído | Must | borrador |
| `HU-021` | Vigencias y estados del documento | Must | borrador |
| `HU-022` | Firma electrónica de niveles 1 y 2 | Should | borrador |

Orden sugerido: `HU-018` → `HU-017` → `HU-019` → `HU-020` → `HU-021` → `HU-022`. El registro de
ejecuciones va primero por la misma razón que la bitácora en `EP-000`: una ejecución sin
registro es una afirmación sin procedencia.

## Dependencias

- **Épicas:** `EP-000` y `EP-001` completas.
- **Preguntas abiertas:** `PA-021` (qué proveedores de IA acepta el cliente, dado que implica
  transferencia internacional de datos), `PA-032` (umbral de confianza que obliga a validación
  humana), `PA-019` (segundo factor para la firma de nivel 2), `PA-020` (proveedor de firma de
  nivel 3), `PA-027` (regla de precedencia entre orígenes).
- **Decisiones:** `ADR-0005` (la IA es un origen, nunca una autoridad), `ADR-0001` (estrategia
  híbrida de extracción, puerto intercambiable), `08-desarrollo/arquitectura-de-aplicacion.md`
  (minimizar lo que se envía al modelo).
- **Supuestos:** `SUP-006` (la IA propone y justifica; la decisión la firma una persona),
  `SUP-004` (alojamiento en Estados Unidos).

## Riesgo abierto

`PA-027` se vuelve exigible aquí. Mientras solo existía el origen `declarado` no había
contradicción posible; a partir de esta épica sí, y hace falta saber qué prevalece y quién puede
cambiarlo. Sin esa respuesta, `HU-019` se puede construir —la discrepancia se muestra igual—
pero no se puede resolver de forma automática en ningún caso, ni siquiera en los evidentes.
