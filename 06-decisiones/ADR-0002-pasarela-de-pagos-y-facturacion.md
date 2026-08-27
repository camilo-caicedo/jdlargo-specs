---
id: ADR-0002
titulo: Cobro de suscripciones, medición de consumo y facturación electrónica
estado: propuesto
fecha: 2026-08-21
---

# ADR-0002 — Pagos, medición y facturación

## Contexto

El cliente definió el modelo comercial: **cada licencia incluye un cupo de consultas
mensuales**; si el cliente lo supera, sube de plan o paga por consulta individual. El
objetivo es controlar precios, gastos y margen. Además, el cobro **entra al piloto**, no
se aplaza (decisión del cliente, revirtiendo la recomendación inicial de `ADR-0001`).

Esto es *usage-based billing con cupo incluido y overage*: el modelo de facturación más
difícil que existe en SaaS, en el mercado que peor lo soporta.

## Hallazgo que determina todo lo demás

Hay que distinguir tres capas que los comparadores comerciales mezclan:

| Capa | Qué significa | Quién la tiene en Colombia |
|---|---|---|
| **Cobro recurrente** | Tokenizar una tarjeta y volver a cobrarla por API | Wompi, ePayco, Mercado Pago |
| **Motor de suscripciones** | Planes, ciclos, prorrateo, trials, dunning | Mercado Pago (limitado), ePayco (básico), Treli |
| **Medición y overage** | Medidores de consumo, cupo incluido, facturación de excedentes | **Nadie con entidad colombiana** |

**Ningún proveedor de pagos colombiano tiene motor de medición de consumo.** Ese es el
hecho central. Dos correcciones adicionales sobre información que circula en blogs:

- **PayU descontinuó su producto de Pagos Recurrentes** y confirma que no será reactivado.
  Los comparadores que lo listan con suscripciones están desactualizados.
- **Stripe no opera con entidad colombiana** (en LATAM solo Brasil y México), y su producto
  *Managed Payments* excluye explícitamente a comercios colombianos.

## Decisión

Tres responsabilidades separadas, ninguna de las cuales se resuelve con un solo proveedor.

### 1. El motor de medición y facturación es propio

No se compra. Se construye sobre las tablas que el proyecto ya necesita para auditoría:

- **Tabla de eventos de consumo** (inmutable): una fila por consulta, con organización,
  costo del proveedor, y referencia a la evidencia. Es la misma tabla de `ADR-0001` §18.
- **Cierre de ciclo mensual** por organización: consumo contra cupo, cálculo de excedente.
- **Tabla de facturas** con su estado.
- **Cuota verificada antes de gastar**, nunca después. Una consulta que excede el cupo se
  bloquea o se marca como excedente de forma explícita, jamás en silencio.

Son dos o tres semanas de trabajo y es el activo que ningún proveedor colombiano entrega.
Comprar Treli (desde ~$329.000 COP/mes) sin haber verificado que su medición hace
exactamente cupo + overage sería gastar antes de saber si se necesita.

### 2. El cobro va por Wompi (Bancolombia), con patrón dual

Wompi por cobertura de métodos locales (PSE, Nequi, Bancolombia, Daviplata), por el peso
comercial del respaldo Bancolombia en ventas B2B colombianas, y porque su Plan Gateway
permite negociar tarifas. Tarifa de referencia: 2,65% + $700 COP + IVA.

- **Personas y clientes pequeños:** tarjeta tokenizada, cobro automático al cierre de ciclo
  por el monto de plan + excedente.
- **Empresas:** no se pelea contra la cultura de pago local. Se emite factura DIAN al inicio
  del ciclo con el consumo del mes anterior y se envía link de pago PSE. Esto además elimina
  el problema del monto variable, que es donde los motores de tarjeta se rompen.

> ⚠️ **Riesgo abierto que hay que resolver antes de construir (`PA-016`).** La documentación
> de Wompi indica que el protocolo 3RI para transacciones automáticas está disponible **solo
> para Mastercard**, y que la activación de 3DS para *payment sources* debe solicitarse al
> equipo de fraude. PSE y Nequi **no son tokenizables** para cobro recurrente. Si Visa no
> funciona para cobro desatendido, una parte grande de los clientes pequeños no es cobrable
> automáticamente. **Es una llamada a Wompi, no una decisión de arquitectura.**

### 3. Facturación electrónica DIAN vía API

Obligatoria: la Resolución 000165 de 2023 cubre a los responsables de IVA y a quienes prestan
servicios. Un cliente empresarial colombiano **necesita la factura electrónica para deducir
el gasto**, así que esto es un requisito comercial, no un trámite.

- **Alegra** — desde ~$17.900 COP/mes (plan Emprendedor), facturas ilimitadas con tope de
  ingresos; ventaja: si el negocio crece, la contabilidad ya está adentro.
- **Factus** — API REST con OAuth2 y sandbox, orientada a desarrolladores; precio no público.

**Decisión: Alegra para el piloto**, revisable si la API de Factus resulta mejor en la prueba.

### 4. Abstracción de método de cobro

La capa de cobro se diseña detrás de un puerto `MetodoDeCobro`, no acoplada a Wompi. Razón
concreta: **Bre-B**, el sistema de pagos inmediatos del Banco de la República, superó los
34 millones de usuarios registrados y los débitos automáticos por llave son la solución
natural al cobro recurrente sin tarjeta en Colombia. A la fecha no están habilitados de
forma nativa, pero cuando lleguen cambian la ecuación completa.

## Consecuencias

**Costo que se asume**

- **Cinco semanas del piloto** se van en medición, cobro y facturación (bloque 4 del
  roadmap). El cliente decidió **no recortar alcance sino extender el calendario**: el piloto
  pasa de 12 a 20 semanas. Ver `02-producto/roadmap.md`.
- El motor propio hay que mantenerlo. A cambio, es también el instrumento que responde
  `PA-011` y `PA-014` con datos reales.
- Ningún proveedor cubre el flujo completo: hay tres integraciones (Wompi, Alegra, y el
  proveedor de screening) en lugar de una.

**A favor**

- El margen queda bajo control desde el primer cliente, que era el objetivo del modelo.
- La misma tabla sirve para evidencia de auditoría y para facturación: un solo activo, dos usos.
- Sin dependencia de un motor de suscripciones extranjero que no admite entidad colombiana.

## Alternativas descartadas

| Alternativa | Por qué no |
|---|---|
| **Entidad en EE.UU. + Stripe Billing** | Stripe Billing es objetivamente superior, pero no tiene **ningún** método de pago colombiano, las tarjetas locales rechazan transacciones internacionales con frecuencia, y una LLC **no puede emitir factura DIAN**, que el cliente empresarial necesita. Además la DIAN tiene régimen de prestadores de servicios desde el exterior: facturar desde afuera no elimina la exposición al IVA colombiano, la traslada a un régimen más incómodo. Válido solo si el mercado real fuera internacional. |
| **PayU como motor de suscripciones** | Producto descontinuado, confirmado por ellos mismos. |
| **Mercado Pago suscripciones** | Tiene motor real de suscripciones, pero **solo monto fijo por ciclo** — no sirve para overage variable — y para suscripciones acepta únicamente tarjeta o dinero en cuenta MP, sin PSE. |
| **Chargebee / Recurly + pasarela local** | Chargebee no integra Wompi, PayU, ePayco ni Mercado Pago para Colombia. Esa combinación no existe. |
| **Paddle como merchant of record** | Su *metered* es un espejismo: no tiene meter API, hay que medir por fuera igual. Además descarta cargos únicos pendientes si la suscripción se cancela — riesgo real de perder el overage de un cliente que se va. Y no tiene PSE. |
| **Lemon Squeezy** | En migración hacia Stripe Managed Payments, que no admite comercios colombianos. |
| **dLocal** | Diseñado para empresas globales cobrando *hacia dentro* de mercados emergentes, no para un SaaS colombiano vendiendo localmente. |
| **Treli** | Único candidato local con motor de suscripciones, pero su medición por consumo no tiene documentación técnica pública. Requiere una llamada técnica antes de comprometer $329.000 COP/mes. No descartado del todo: reevaluar si el motor propio resulta más caro de lo previsto. |

## Pendientes antes de construir

1. **Llamar a Wompi** y confirmar cobro desatendido con Visa, no solo Mastercard (`PA-016`).
2. Pedir tarifas por método (PSE, Nequi, Bancolombia por separado) — solo publican la general.
3. Confirmar con un contador colombiano el tratamiento de IVA sobre el servicio y sobre los
   excedentes facturados.
4. Probar la API de Alegra en sandbox antes de cerrar la decisión.

## Fuentes

- Wompi 3DS/3RI: <https://docs.wompi.co/en/docs/colombia/fuentes-de-pago-3ds/>
- Wompi tarifas: <https://wompi.com/es/co/planes-tarifas/plan-avanzado-agregador>
- PayU deprecación: <https://developers.payulatam.com/latam/es/deprecated/recurring-payments/recurring-payments-api.html>
- Stripe disponibilidad: <https://stripe.com/global>
- Stripe Managed Payments elegibilidad: <https://docs.stripe.com/payments/managed-payments/eligibility>
- Mercado Pago suscripciones CO: <https://www.mercadopago.com.co/developers/es/docs/subscriptions/overview>
- Resolución DIAN 000165 de 2023: <https://normograma.dian.gov.co/dian/compilacion/docs/resolucion_dian_0165_2023.htm>
- Alegra precios: <https://www.alegra.com/colombia/facturacion-electronica/precios/>
- Factus API: <https://developers.factus.com.co/>
