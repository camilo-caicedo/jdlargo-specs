---
id: EP-007
titulo: Capa comercial
estado: borrador
capacidad: CAP-07
actualizado: 2026-09-05
---

# EP-007 — Capa comercial

## Objetivo

Cobrar por el producto: planes con cupo de consultas incluido, medición fiable del consumo, cobro
del excedente y factura electrónica válida en Colombia.

## Por qué existe

Es la fase **C** del `02-producto/roadmap.md`, la única pieza del plan **sin lugar fijo**: va donde
aparezca el primer cliente que pague. Si Juan David opera como cliente ancla facturado a mano,
puede ir después de la Fase 4; si se abre registro público antes, hay que adelantarla.

> **Actualización 2026-09-07.** Esto ya ocurrió: el *mecanismo* de registro público se
> adelantó al MVP (`HU-060`, `EP-001`, decisión de Camilo). Lo que **no** se adelantó es esta
> épica — sigue bloqueada por `PA-043` (precios). Una organización que se registra sola por
> `HU-060` no tiene plan ni cupo asignado hasta que esta épica exista; es una brecha
> deliberada, documentada, no un olvido.

`ADR-0002` ya hizo el trabajo difícil y dejó tres conclusiones que esta épica implementa:

1. **El motor de medición y facturación es propio.** Ningún proveedor de pagos colombiano tiene
   motor de medición de consumo con cupo incluido y excedente. Se construye sobre las tablas que el
   proyecto ya necesita para auditoría.
2. **El cobro va por Wompi, por facturación con enlace de pago** (PSE, tarjeta, Efecty o
   transferencia), para todo cliente por igual. `PA-016` cerró a favor de esta única vía: se
   descartó la tarjeta tokenizada con débito automático que este documento traía como patrón
   dual, porque obligaba a registrar tarjeta y el cobro desatendido de Wompi solo está
   documentado para Mastercard.
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
- Cobro por pasarela, vía única de factura más enlace de pago (`HU-049`).
- Factura electrónica (`HU-050`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Selección de plan, cobro y verificación KYB al registrarse | **Fuera del MVP**, bloqueado por `PA-043` | El *mecanismo* de registro ya está en el MVP (`HU-060`); esta capa comercial alrededor sigue pendiente |
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
cobrar, y porque es también el instrumento que permitirá fijar los precios de `PA-043` con datos
reales de consumo.

## Dependencias

- **Épicas:** `EP-000` (aislamiento y bitácora) y `EP-003` (es donde se genera el consumo).
- **Preguntas abiertas:** **`PA-043`** (precios y cupos concretos de cada plan) y **`PA-040`**
  (costo real por consulta, del que depende `PA-043`). Resueltas: `PA-016` → **no hay débito
  automático**, se factura y se envía link de pago (`ADR-0002` §2); `PA-015` y `PA-036` → modelo
  híbrido de suscripción + créditos + módulos; `PA-037` → excedente facturable en
  Professional/Enterprise, bloqueo con *upgrade* en Starter, alertas al 80/90/100 %; `PA-038` →
  alta manual al principio, autoservicio después — **el mecanismo se adelantó al MVP
  (`HU-060`), esta capa comercial sigue igual de bloqueada**; `PA-014` → `RNF-019`;
  `PA-035` → `ADR-0009`.
- **Decisiones:** `ADR-0002` entera.

## Riesgo abierto

~~**`PA-016` es bloqueante**~~ — **resuelta, y desactivada por la vía más limpia**: el cliente
descartó el débito automático con tarjeta, así que la pregunta dejó de importar. El análisis
original se conserva porque explica qué se evitó: el cobro desatendido de Wompi solo está
documentado para Mastercard, así que una parte grande de los clientes pequeños no habría sido
cobrable de forma automática. En vez de mantener el patrón dual como respaldo para ese caso, el
cliente lo descartó entero — factura más enlace de pago es la única vía, para todos.

El segundo riesgo es que el margen **sigue sin poderse calcular**: `PA-014` y `PA-035` ya están
respondidas, pero falta `PA-040` (cuánto
frecuencia se re-consultan). Fijar precios de plan antes de tener esos tres números es fijarlos a
ciegas.

`ADR-0002` deja además cuatro pendientes concretos antes de construir: llamar a Wompi, pedir
tarifas por método de pago, confirmar el tratamiento del impuesto sobre el servicio y sobre los
excedentes con un contador, y probar la interfaz de Alegra en su entorno de pruebas.
