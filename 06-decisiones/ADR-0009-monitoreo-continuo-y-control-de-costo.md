---
id: ADR-0009
titulo: Monitoreo continuo por riesgo y por evento, con control de costo
estado: propuesto
fecha: 2026-09-05
reemplaza-a: —
reemplazado-por: —
---

# ADR-0009 — Monitoreo continuo y control del costo variable

## Contexto

El monitoreo continuo quedó **confirmado en alcance** (`PA-011`) y es, a la vez, el
diferenciador central del modelo SaaS y **la principal fuente de costo variable del producto**
(`PA-035`). Las dos cosas son ciertas al mismo tiempo, y por eso esto es un ADR y no una
historia más.

Los números que hay sobre la mesa:

- Un cliente de transporte de carga hace ~**1.000 consultas al mes** (`PA-014`).
- Costo de referencia por consulta: **$1.000–$2.000 COP**, provisional (`PA-012`, `PA-040`).
- Capacidad objetivo del MVP: **10.000–50.000 contrapartes por organización cliente** sin
  rediseño (`PA-014`).

Re-escanear ciegamente 50.000 contrapartes cada mes son 50.000 consultas. Con tarifa plana,
ese cliente deja el negocio en pérdida y nadie se entera hasta que llega la factura.

## Opciones consideradas

### Opción A — Periodicidad fija e igual para todos
- A favor: trivial de explicar y de programar.
- En contra: paga lo mismo por vigilar a una contraparte de riesgo bajo que a un PEP. Es la
  forma más cara de obtener el peor resultado, y además no responde al enfoque basado en
  riesgo que exige la norma.

### Opción B — Periodicidad por riesgo + re-screening por evento
- A favor: el gasto sigue al riesgo; permite reaccionar el mismo día ante un cambio en una
  fuente; es lo que la norma pide de verdad.
- En contra: exige planificador, colas, deduplicación y un catálogo de eventos por fuente.
  Bastante más máquina.

### Opción C — Solo aviso al usuario de que "toca monitorear"
- A favor: costo variable cercano a cero; es lo que el cliente propuso como arranque.
- En contra: no es monitoreo, es un recordatorio. No detecta nada por sí solo.

## Decisión

**Opción B como destino, con la C dentro de ella como modo de operación configurable.**

### 1. Tres disparadores, no uno

| Disparador | Qué lo activa | Cuándo |
|---|---|---|
| **Por periodicidad** | El nivel de riesgo de la contraparte | Programado |
| **Por evento** | Un cambio relevante en la fuente (nueva sanción, cambio societario, de representante legal o de beneficiario final) | Inmediato, donde el proveedor lo permita |
| **Por aviso al usuario** | La contraparte entra en ventana de revisión | Correo o alerta al responsable, que decide |

El tercero es el que pidió el cliente como arranque, y se conserva **como modo configurable**:
hay organizaciones que preferirán autorizar cada lote de consultas antes de gastarlo.

### 2. Periodicidad de referencia, no obligación universal

| Riesgo | Frecuencia de referencia |
|---|---|
| Alto | Mensual o trimestral |
| Medio | Semestral |
| Bajo | Anual |

**Son valores por defecto de la configuración, no una promesa comercial ni una regla legal.**
La frecuencia concreta sale del enfoque basado en riesgo del cliente y del marco aplicable
(`ADR-0004`). No se venden estos plazos como si la norma los fijara.

### 3. Cuatro controles de costo, desde el día uno

1. **Planificador por organización, riesgo y fuente** — no un barrido global.
2. **Deduplicación por identidad**: no se consulta dos veces al mismo sujeto dentro de una
   ventana, aunque aparezca en varios expedientes de esa organización.
3. **Re-screening incremental**: evaluar contra lo que cambió en el conjunto de datos, no
   lanzar una llamada por contraparte. Es, por sí solo, el principal argumento a favor del
   auto-hospedaje del índice (`ADR-0003`).
4. **Cuota verificada antes de gastar**, nunca después (`ADR-0002` §2b). Con una excepción
   dura: **un monitoreo en curso no se corta en silencio por un límite** — se avisa, se ofrece
   excedente o *upgrade*, y se escala.

### 4. Todo cambio detectado es una afirmación nueva

Un evento de monitoreo no sobrescribe el expediente: crea una afirmación que contradice a la
anterior y, si procede, una alerta y un caso (`ADR-0005`). El expediente cerrado sigue siendo
inmutable.

## Consecuencias

- **Positivas:** el costo del monitoreo se vuelve proporcional al riesgo y medible por
  organización; se puede ofrecer con niveles de servicio distintos por plan; la reacción ante
  un cambio deja de depender de que alguien se acuerde.
- **Negativas / costo asumido:** planificador, colas, deduplicación y catálogo de eventos son
  trabajo real que no produce pantalla visible. Y el re-screening incremental depende de que
  el proveedor entregue *deltas* o *webhooks*: si no lo hace, se degrada a barrido periódico y
  el costo sube. Hay que confirmarlo antes de prometer el nivel de servicio.
- **Qué habría que revisar si el contexto cambia:** el punto de equilibrio entre API SaaS y
  auto-hospedaje se cruza justo con el monitoreo. Cuando el volumen mensual lo supere, se abre
  `ADR-0003`.

## Trazabilidad

- Cierra `PA-011` (confirmación de alcance), `PA-014` y `PA-035`.
- Afecta: `HU-040`, `HU-041`, `HU-042`, `HU-025`, `HU-047`, `07-integraciones/README.md`,
  `ADR-0002`.
