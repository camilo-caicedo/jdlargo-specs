---
id: ADR-0008
titulo: El matching es multicriterio y la confirmación es humana
estado: propuesto
fecha: 2026-09-05
reemplaza-a: —
reemplazado-por: —
---

# ADR-0008 — Comparación de nombres: multicriterio, no un porcentaje

## Contexto

`PA-033` preguntaba qué umbral de similitud dispara una coincidencia en el matching de
nombres. La pregunta llevaba dentro una trampa: da por hecho que existe un número que separa
lo que es de lo que no es.

No existe. Un umbral bajo inunda al analista de falsos positivos hasta que deja de mirarlos —
que es la peor forma de fallar, porque el control parece existir. Un umbral alto deja pasar
coincidencias reales. Y ninguno de los dos resuelve el problema de fondo: en listas
restrictivas, dos personas distintas comparten nombre con muchísima más frecuencia de lo que
sugiere la intuición, y una misma persona aparece escrita de cinco maneras.

## Opciones consideradas

### Opción A — Un umbral único de similitud (p. ej. 85 %)
- A favor: trivial de implementar y de explicar.
- En contra: "85 % = positivo" es falso. Ignora identificadores, fecha de nacimiento, país y
  alias, que es justo la información que decide. Convierte una heurística en un veredicto.

### Opción B — Score multicriterio con zonas y confirmación humana
- A favor: es como funciona el screening serio; permite subir la sensibilidad sin ahogar al
  analista; los umbrales se ajustan por fuente, que es donde varía la calidad del dato.
- En contra: más trabajo de implementación, y exige explicar al usuario por qué algo quedó en
  zona de candidato.

### Opción C — Confirmación automática por encima de cierto score
- A favor: elimina trabajo humano.
- En contra: inaceptable. Marcar automáticamente a alguien como sancionado es la decisión de
  mayor impacto que toma el sistema, y es exactamente la que no le corresponde.

## Decisión

**Opción B.**

### 1. Tres zonas, no dos

| Zona | Qué es | Qué hace el sistema |
|---|---|---|
| **Descarte** | Por debajo del umbral de generación | No crea coincidencia; queda registrado que se consultó |
| **Candidato** | Desde ~85 % de similitud de nombre, como referencia inicial | Crea coincidencia en estado `posible` y la manda a revisión |
| **Confirmada** | Solo tras validación humana | Estado `confirmada`, con quién y por qué |

**El sistema nunca pasa por sí solo de `posible` a `confirmada`.** La confirmación es siempre
un acto humano registrado (`ADR-0005` §3).

### 2. El score no es solo el nombre

Entran al cálculo, con peso propio: **identificador** (cédula, NIT, pasaporte), **alias**,
**fecha de nacimiento**, **país o jurisdicción** y **transliteración** de nombres no latinos.
Un identificador que coincide vale más que un nombre idéntico; un país que no coincide baja el
score aunque el nombre calce al 100 %.

### 3. Los umbrales se configuran por fuente

No es lo mismo la calidad de OFAC-SDN que la de un listado local sin identificadores. Cada
fuente del catálogo (`HU-023`) lleva sus propios umbrales de descarte y de candidato, y son
configuración versionada (`ADR-0004`), no constantes en el código.

### 4. Se aprende de los falsos positivos, sin reescribir el pasado

Los descartes confirmados por una persona alimentan una lista de exclusión por organización
cliente, para no volver a preguntar lo mismo. **Ese aprendizaje nunca altera retroactivamente
una decisión ya tomada**: el expediente conserva la coincidencia que vio y cómo se resolvió.

## Consecuencias

- **Positivas:** el analista revisa menos ruido sin perder sensibilidad; el sistema puede
  explicar por qué algo es candidato, que es lo que un supervisor pregunta; la calidad
  desigual de las fuentes deja de contaminar el resultado global.
- **Negativas / costo asumido:** es notablemente más caro de construir y de afinar que un
  porcentaje, y exige una pantalla de revisión buena — con la evidencia lado a lado — o la
  ventaja se pierde. La lista de exclusión hay que auditarla: es un lugar donde un descarte
  equivocado se vuelve permanente y silencioso.
- **Qué habría que revisar si el contexto cambia:** si se adopta un proveedor que ya entrega
  scoring propio, esta decisión pasa a ser cómo se traduce **su** score a estas tres zonas, no
  cómo se calcula.

## Trazabilidad

- Cierra `PA-033`.
- Afecta: `HU-025`, `HU-026`, `HU-027`, `HU-028`, `HU-023`, `00-contexto/glosario.md`.
