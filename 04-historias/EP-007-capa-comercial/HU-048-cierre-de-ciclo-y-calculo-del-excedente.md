---
id: HU-048
titulo: Cierre de ciclo y cálculo del excedente
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-08-27
---

# HU-048 — Cierre de ciclo y cálculo del excedente

## Historia

**Como** proveedor del servicio
**quiero** cerrar cada mes el consumo de cada cliente y calcular qué parte excede su cupo
**para** poder cobrar lo que corresponde con un cálculo que puedo explicar consulta por consulta.

## Contexto

Es el segundo componente del motor propio de `ADR-0002`: **cierre de ciclo mensual por
organización, consumo contra cupo, cálculo de excedente**. Sin él, la medición de `HU-047` es
información sin consecuencia.

El requisito no evidente es que el cierre tiene que ser **explicable y reproducible**. Un cliente
que recibe un cargo por excedente va a preguntar de dónde sale, y la respuesta tiene que ser una
lista de consultas con fecha, fuente y expediente, no un total.

Y tiene que ser **inmutable una vez cerrado**: recalcular un ciclo pasado porque cambió el plan o
apareció una consulta tardía es la vía directa a facturas que no cuadran con lo que ya se cobró.

## Criterios de aceptación

```gherkin
Escenario: Cerrar un ciclo mensual
  Dado una organización cliente con consumo en el ciclo en curso
  Cuando llega la fecha de cierre
  Entonces se cierra el ciclo con el total de consultas, el cupo incluido y el excedente calculado
  Y el ciclo queda inmutable
  Y se abre el ciclo siguiente
```

```gherkin
Escenario: El excedente se calcula contra el cupo del plan vigente
  Dado un ciclo con más consultas de las que incluye el plan
  Cuando se cierra
  Entonces el excedente es la diferencia entre lo consumido y el cupo del plan vigente durante ese ciclo
  Y se valora al precio de excedente de ese mismo plan
```

```gherkin
Escenario: El cálculo se puede desglosar
  Dado un ciclo cerrado con excedente
  Cuando se consulta su detalle
  Entonces se obtiene la lista de consultas que lo componen, con su fecha, su fuente y su expediente
  Y la suma coincide exactamente con el total cobrado
```

```gherkin
Escenario: Un ciclo cerrado no se recalcula
  Dado un ciclo ya cerrado
  Cuando aparece una consulta tardía o cambia el plan del cliente
  Entonces el ciclo cerrado no se modifica
  Y lo que corresponda se refleja en el ciclo siguiente, con su explicación
```

```gherkin
Escenario: Un ciclo sin excedente también se cierra
  Dado una organización cliente que consumió menos que su cupo
  Cuando cierra el ciclo
  Entonces el ciclo se cierra con excedente cero
  Y el cupo no consumido no se acumula al ciclo siguiente
```

```gherkin
Escenario: El cierre es reintentable sin duplicar
  Dado un proceso de cierre que falla a mitad
  Cuando se reintenta
  Entonces no se generan dos cierres para el mismo ciclo
  Y el resultado es idéntico al que habría producido la primera ejecución
```

```gherkin
Escenario: El margen es calculable
  Dado un ciclo cerrado
  Cuando se consulta su resultado
  Entonces se obtiene el ingreso del ciclo, el costo de proveedores incurrido y la diferencia
  Y el costo sale de los eventos de consumo, no de una estimación
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los ciclos
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta sus ciclos con su contexto de usuario propagado
  Entonces obtiene únicamente los suyos
```

## Reglas de negocio

- El ciclo es **mensual y por organización cliente**. Se cierra en su fecha y queda **inmutable**.
- El excedente se calcula contra el cupo del **plan vigente durante ese ciclo**, no el actual.
- El cierre debe poder **desglosarse consulta por consulta**, y la suma del desglose debe coincidir
  con el total. Un total que no se puede desglosar no se cobra.
- **Un ciclo cerrado no se recalcula.** Lo que llegue tarde se refleja en el ciclo siguiente, con
  su explicación.
- El cupo no consumido **no se acumula** al ciclo siguiente, salvo que el plan diga lo contrario.
- El proceso de cierre es idempotente y reintentable.
- Del cierre sale el **margen** del ciclo: ingreso menos costo real de proveedores, tomado de los
  eventos de consumo (`HU-047`).

## Fuera de alcance

- El cobro del importe resultante → `HU-049`.
- La emisión de la factura → `HU-050`.
- El prorrateo por cambio de plan a mitad de ciclo: no está definido y no se inventa.
- La acumulación de cupo no consumido entre ciclos, mientras el cliente no la pida.
- Descuentos, promociones y condiciones especiales negociadas.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `ciclo.organization_id` | Sí | Organización cliente existente | No |
| `ciclo.desde` / `hasta` | Sí | Periodo mensual; sin solapamiento con otro ciclo | No |
| `ciclo.estado` | Sí | `abierto` \| `cerrado` | No |
| `ciclo.plan_version_id` | Sí | Plan vigente durante el ciclo | No |
| `ciclo.consultas_totales` | Sí | Igual al número de eventos de consumo del periodo | No |
| `ciclo.cupo_incluido` | Sí | Del plan vigente | No |
| `ciclo.excedente` | Sí | Consultas por encima del cupo; cero o positivo | No |
| `ciclo.importe_excedente` | Sí | Excedente por el precio de excedente del plan | No |
| `ciclo.costo_proveedores` | Sí | Suma de los costos de los eventos de consumo | No |
| `ciclo.cerrado_en` | Condicional | Momento del cierre; se escribe una sola vez | No |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` (cierre de ciclo mensual, cálculo de excedente, motor propio)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-036` (cupos y precios), `PA-037` (si se permite excedente o se
  bloquea, que determina si esta historia tiene algo que calcular). **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-046` (planes), `HU-047` (eventos de consumo).
- **Habilita a:** `HU-049`, `HU-050`, y el seguimiento real del margen por cliente.
- **Riesgo:** recalcular ciclos cerrados es una tentación razonable —"es que llegó tarde una
  consulta"— y produce facturas que no cuadran con lo cobrado. Conviene que la inmutabilidad del
  ciclo esté impuesta en la base de datos y no en la disciplina de quien opera.
