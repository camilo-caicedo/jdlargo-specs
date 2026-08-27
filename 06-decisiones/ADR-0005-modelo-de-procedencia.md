---
id: ADR-0005
titulo: Todo dato del expediente lleva su procedencia
estado: propuesto
fecha: 2026-08-25
---

# ADR-0005 — Modelo de procedencia del dato

## Contexto

La §2 del [documento del cliente](../01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md)
es, en sus propias palabras, "la idea más importante para entender el resto":

| # | Concepto | Significa |
|---|---|---|
| 1 | **Recolección** | Lo dijo la contraparte |
| 2 | **Extracción** | Lo leyó la IA de un documento |
| 3 | **Verificación** | Lo confirmó una fuente externa autorizada |
| 4 | **Evaluación** | El sistema aplicó una regla |
| 5 | **Decisión** | Una persona autorizada, con nombre y cargo, decidió |

> "Nunca deben mezclarse ni presentarse como si fueran lo mismo."

Esto parece una regla de presentación y **es una restricción de modelo de datos**. Un
esquema convencional —una tabla `contraparte` con una columna `nombre`— no puede cumplirla:
en el momento en que se escribe un valor en esa columna, se perdió de dónde vino, y con
ello la Fase 9 (conciliación), la Fase 8 (la IA nunca sobrescribe en silencio) y el criterio
de aceptación entero.

## Decisión

### 1. El expediente no guarda valores: guarda afirmaciones

Cada dato es una **afirmación** con su procedencia. Un mismo campo puede tener varias
afirmaciones simultáneas y contradictorias, y eso no es un error: es el estado normal de un
expediente en curso.

Cada afirmación guarda: expediente, sujeto, campo, valor, **origen** (declarado · extraído ·
verificado · evaluado), quién o qué la produjo, cuándo, evidencia asociada, nivel de
confianza (si vino de un modelo), y estado.

Ejemplo del propio documento (Fase 9):

```
razon_social · "ABC S.A.S."           · declarado  · contraparte      · —
razon_social · "ABC LOGÍSTICA S.A.S." · verificado · Cámara de Comercio · doc#8821
→ discrepancia abierta, revisión humana requerida
```

**Nunca se sobrescribe una afirmación.** Se añade otra, y la discrepancia queda visible
hasta que una persona la resuelva dejando registro.

### 2. El "valor vigente" es un cálculo, no una columna

Cuál de las afirmaciones prevalece sale de una regla de precedencia configurable por cliente
(`ADR-0004`) — típicamente lo verificado pesa más que lo declarado, y lo declarado más que
lo extraído. Pero la afirmación descartada **no se borra**: sigue siendo evidencia de que
hubo una diferencia.

### 3. Evaluaciones y decisiones son eventos, no estados

- Una **evaluación** (riesgo, regla, screening) se persiste con sus entradas, la versión de
  la metodología, el resultado y las reglas que se dispararon. Recalcular no la reemplaza:
  crea otra.
- Una **decisión** lleva siempre persona identificada, cargo, fundamento, evidencia en la
  que se basó, condiciones y vigencia (Fase 17). Es inmutable.

### 4. La IA es un origen más, nunca una autoridad

Toda ejecución de IA registra modelo, proveedor, versión, plantilla de instrucciones,
documento fuente, resultado, confianza y quién lo validó (§32). Una afirmación de origen
`extraído` **no puede** ascender a `verificado` sin una acción humana o una fuente externa
que lo respalde.

### 5. Una sola bitácora inmutable, transversal

El registro de auditoría (Fase 23) no es un módulo: es el sustrato. Quién, qué, cuándo,
desde dónde, valor anterior y nuevo, motivo, fuente, si fue automático o manual, qué modelo
intervino y qué versión de regla estaba vigente.

## Consecuencias

**A favor**

- Las 16 preguntas del criterio de aceptación (§44) se responden **por construcción**, no
  con un reporte que hay que fabricar aparte.
- La conciliación de la Fase 9 deja de ser una función: es una consulta sobre afirmaciones
  del mismo campo con distinto origen.
- Es imposible que la IA "limpie" un expediente en silencio: no existe la operación de
  sobrescribir.
- La misma estructura sirve para el monitoreo continuo: un cambio detectado en la Fase 20 es
  una afirmación nueva que contradice a una anterior.

**Costo que se asume**

- **Consultar es más caro y más incómodo** que en un modelo plano. Se mitiga con vistas
  materializadas del "valor vigente" por expediente, reconstruibles y nunca autoritativas.
- Todo formulario y toda pantalla tienen que pensarse en términos de afirmaciones, no de
  campos. Es un cambio de mentalidad más que de tecnología, y es donde más se equivoca quien
  llega con costumbres de CRUD.
- Volumen de filas mucho mayor. Irrelevante a la escala prevista (menos de 100 organizaciones).

## Alternativas descartadas

| Alternativa | Por qué no |
|---|---|
| Tabla plana + bitácora de cambios aparte | La bitácora responde "qué cambió", no "qué dice cada fuente ahora". La conciliación se vuelve imposible de consultar. |
| Columnas paralelas (`nombre_declarado`, `nombre_extraido`, `nombre_verificado`) | No escala a campos configurables (`ADR-0004`) ni admite varias fuentes verificadoras del mismo campo. |
| Guardar solo el valor final y confiar en el registro de IA | Es exactamente lo que el documento prohíbe: la diferencia desaparece del expediente. |
