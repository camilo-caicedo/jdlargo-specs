---
id: EP-007
titulo: Capa comercial
estado: borrador
capacidad: CAP-07
actualizado: 2026-08-27
---

# EP-007 — Capa comercial

## Objetivo

Cobrar por el producto: planes con cupo de consultas incluido, medición fiable del consumo, cobro
del excedente y factura electrónica válida en Colombia.

## Por qué existe

Es la fase **C** del `02-producto/roadmap.md`, la única pieza del plan **sin lugar fijo**: va donde
aparezca el primer cliente que pague. Si Juan David opera como cliente ancla facturado a mano,
puede ir después de la Fase 4; si se abre registro público antes, hay que adelantarla.

`ADR-0002` ya hizo el trabajo difícil y dejó tres conclusiones que esta épica implementa:

1. **El motor de medición y facturación es propio.** Ningún proveedor de pagos colombiano tiene
   motor de medición de consumo con cupo incluido y excedente. Se construye sobre las tablas que el
   proyecto ya necesita para auditoría.
2. **El cobro va por Wompi**, con patrón dual: tarjeta tokenizada para clientes pequeños, factura
   más enlace de pago para empresas.
3. **La factura electrónica es obligatoria** y va por interfaz de programación, con Alegra en el
   piloto.

Y una advertencia del propio `ADR-0002` que conviene tener presente al planificar: esto es
*"usage-based billing con cupo incluido y overage: el modelo de facturación más difícil que existe
en SaaS, en el mercado que peor lo soporta"*.

## Alcance

**Incluye:**

- Planes, licencias y cupo de consultas (`HU-046`).
- Medición de consumo y control del cupo antes de gastar (`HU-047`).
- Cierre de ciclo y cálculo del excedente (`HU-048`).
- Cobro por pasarela, con patrón dual (`HU-049`).
- Factura electrónica (`HU-050`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Registro público de clientes nuevos por autoservicio | Depende de `PA-038` | Si el alta es siempre manual, no hace falta |
| Motor de suscripciones de terceros | — | `ADR-0002` lo descarta: ninguno cubre cupo más excedente en Colombia |
| Pagos con débito automático por llave del sistema local de pagos inmediatos | Cuando exista | `ADR-0002` deja el puerto listo; hoy no está habilitado |
| Prorrateo, periodos de prueba y gestión de impagos elaborada | Fase posterior | El piloto no lo necesita |
| Contabilidad completa | — | Alegra la cubre si el negocio crece; no es alcance del producto |

## Actores involucrados

- **Cliente del SaaS** — contrata el plan y paga.
- **Administrador** — ve su consumo y su cupo.
- **Sistema** — mide, cierra el ciclo, cobra y factura.
- **Nosotros como proveedores** — administramos planes y conciliamos.

## Criterios de éxito

1. **La cuota se verifica antes de gastar**, nunca después.
2. **Cero consumo sin evidencia y cero evidencia sin consumo**: la misma fila sirve para las dos
   cosas.
3. **Cero cobros duplicados**: todo evento de pasarela es idempotente.
4. Un cliente empresarial recibe una **factura electrónica válida** que le permite deducir el
   gasto.
5. El margen por cliente es **visible y calculable** en cualquier momento: consumo, costo de
   proveedores e ingreso.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-046` | Planes, licencias y cupo de consultas | Must | borrador |
| `HU-047` | Medición de consumo y control del cupo | Must | borrador |
| `HU-048` | Cierre de ciclo y cálculo del excedente | Must | borrador |
| `HU-049` | Cobro por pasarela | Must | borrador |
| `HU-050` | Factura electrónica | Must | borrador |

Orden sugerido: el de la lista. La medición va antes que el cobro porque sin ella no hay qué
cobrar, y porque es también el instrumento que responde `PA-011` y `PA-014` con datos reales.

## Dependencias

- **Épicas:** `EP-000` (aislamiento y bitácora) y `EP-003` (es donde se genera el consumo).
- **Preguntas abiertas:** `PA-016` (**bloqueante**: si Wompi permite cobro desatendido con Visa o
  solo con Mastercard), `PA-036` (planes, precios y cupos), `PA-037` (qué pasa al agotar el cupo),
  `PA-038` (alta por autoservicio o manual), `PA-014` (cuántas contrapartes por cliente),
  `PA-012` (costo de los proveedores), `PA-035` (cuánto consume el monitoreo).
- **Decisiones:** `ADR-0002` entera.

## Riesgo abierto

**`PA-016` es bloqueante y es una llamada de teléfono, no una decisión de arquitectura.** Si el
cobro desatendido solo funciona con Mastercard, una parte grande de los clientes pequeños no es
cobrable de forma automática, y el patrón dual de `ADR-0002` deja de ser una preferencia para
convertirse en la única vía.

El segundo riesgo es que el margen **no se puede calcular todavía**: depende de `PA-012` (cuánto
cuesta cada consulta), `PA-014` (cuántas contrapartes tiene un cliente) y `PA-035` (con qué
frecuencia se re-consultan). Fijar precios de plan antes de tener esos tres números es fijarlos a
ciegas.

`ADR-0002` deja además cuatro pendientes concretos antes de construir: llamar a Wompi, pedir
tarifas por método de pago, confirmar el tratamiento del impuesto sobre el servicio y sobre los
excedentes con un contador, y probar la interfaz de Alegra en su entorno de pruebas.
