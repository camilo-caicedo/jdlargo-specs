---
id: ADR-0002
titulo: Cobro de suscripciones, medición de consumo y facturación electrónica
estado: propuesto
fecha: 2026-08-21
actualizado: 2026-09-05
---

# ADR-0002 — Pagos, medición y facturación

> **Actualización 2026-09-05.** Las respuestas del cliente a `PA-015`, `PA-016`, `PA-036`,
> `PA-037` y `PA-038` cambian la decisión de cobro: **se elimina el débito automático con
> tarjeta tokenizada**. El cliente decidió cobrar por facturación, con el medio de pago que
> cada quien elija. Eso desactiva el riesgo bloqueante que este ADR tenía abierto (Wompi 3RI
> solo para Mastercard) sin necesidad de resolverlo. El motor propio de medición, la
> facturación DIAN y la abstracción de método de cobro se mantienen tal cual.

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

### 2. El cobro es por facturación, no por débito automático

**Decisión del cliente al cerrar `PA-016`:** no se le pide a nadie registrar una tarjeta para
débito automático. Se factura, y el cliente paga con el medio que prefiera.

Un solo patrón para todos, sin distinguir persona de empresa:

1. Se cierra el ciclo y se calcula plan + excedente.
2. Se emite la **factura electrónica DIAN**.
3. Se envía la factura con un **link de pago**, y el cliente elige medio:
   **PSE · tarjeta débito/crédito · Efecty · transferencia** a la cuenta bancaria del negocio.
4. La plataforma concilia el pago contra la factura y libera el ciclo siguiente.

Wompi (Bancolombia) sigue siendo la pasarela para PSE, Nequi, Bancolombia, Daviplata y
tarjeta, ahora en modo **pago único por link**, que es su caso soportado sin reservas. Tarifa
de referencia: 2,65 % + $700 COP + IVA. La transferencia directa y Efecty se concilian a
mano o por conciliación bancaria; no todas las vías pasan por la pasarela.

> ✅ **Riesgo cerrado.** El bloqueo anterior (`PA-016`: 3RI de Wompi disponible solo para
> Mastercard, PSE y Nequi no tokenizables) **deja de importar**, porque ya no hay cobro
> desatendido. Se resolvió eliminando la dependencia, no llamando a Wompi. La llamada sigue
> siendo útil solo para negociar tarifas por método.

**Costo que esto traslada:** la cobranza pasa a ser un proceso con intervención humana —
recordatorios, conciliación de transferencias, mora. A cambio se elimina el fallo más caro
del modelo anterior: la parte del mercado que simplemente no era cobrable de forma automática.

### 2b. Planes, cupos y excedentes

Cierra `PA-015`, `PA-036` y `PA-037`. **Los planes no se codifican**: son configuración.

Modelo **híbrido**, no "licencia" como métrica única:

```
suscripción base (por organización cliente)
  + usuarios incluidos
  + cupo de consultas / créditos
  + módulos premium
  + excedente por consumo
```

Tres planes de referencia — **Starter · Professional · Enterprise** — con límites por
usuarios, contrapartes, consultas, OCR/IA y monitoreo. El precio se calcula sobre costo
variable real + margen; las cifras concretas quedan en `PA-043`, no aquí.

Comportamiento al agotar el cupo (`PA-037`):

| Plan | Al llegar al 100 % del cupo |
|---|---|
| **Starter** | Bloqueo con propuesta de *upgrade* o *top-up*. Nunca en silencio |
| **Professional / Enterprise** | **Excedente facturable** con período de gracia, autorizado por contrato |

Reglas duras, para todos los planes:

- **Alertas de consumo al 80 %, 90 % y 100 %**, antes de que el límite muerda.
- El excedente debe estar declarado en el contrato y visible en la plataforma **antes** de
  generarse. Nadie se entera de un cobro adicional en la factura.
- **Nunca se corta un control crítico de cumplimiento por un límite silencioso.** Un
  monitoreo en curso no se interrumpe sin aviso previo; ese es el peor fallo posible del
  módulo comercial.
- **Metering en tiempo real**: consultas, screening, OCR/IA, documentos y usuarios, con
  panel de consumo para el cliente.

### 2c. Alta de organizaciones cliente

Cierra `PA-038`. **Al principio, alta manual de nuestra parte.** Después, híbrido:
autoservicio con KYB/validación automática para planes bajos, onboarding asistido para
Enterprise. La propia empresa cliente pasa por un onboarding comercial/KYB — que es un flujo
distinto de la debida diligencia que ella aplica a sus terceros (`PA-004`).

Consecuencia de alcance: **el MVP no necesita registro público** ni pasarela de autoservicio
en la primera fase. Eso descarga la capa comercial del roadmap.

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
- El motor propio hay que mantenerlo. A cambio, es también el instrumento que da cifras
  reales de consumo, que es lo único que permitirá fijar los precios de `PA-043`.
- **La cobranza es manual.** Sin débito automático hay recordatorios, conciliación y mora.
  Es trabajo operativo recurrente que antes no estaba en el plan.
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
| **Tarjeta tokenizada con cobro automático** | Descartada por el cliente (`PA-016`): obliga a registrar tarjeta, choca con la cultura de pago local en B2B, y en Wompi el cobro desatendido solo está documentado para Mastercard. Se cambia por factura + link de pago. |
| **Chargebee / Recurly + pasarela local** | Chargebee no integra Wompi, PayU, ePayco ni Mercado Pago para Colombia. Esa combinación no existe. |
| **Paddle como merchant of record** | Su *metered* es un espejismo: no tiene meter API, hay que medir por fuera igual. Además descarta cargos únicos pendientes si la suscripción se cancela — riesgo real de perder el overage de un cliente que se va. Y no tiene PSE. |
| **Lemon Squeezy** | En migración hacia Stripe Managed Payments, que no admite comercios colombianos. |
| **dLocal** | Diseñado para empresas globales cobrando *hacia dentro* de mercados emergentes, no para un SaaS colombiano vendiendo localmente. |
| **Treli** | Único candidato local con motor de suscripciones, pero su medición por consumo no tiene documentación técnica pública. Requiere una llamada técnica antes de comprometer $329.000 COP/mes. No descartado del todo: reevaluar si el motor propio resulta más caro de lo previsto. |

## Pendientes antes de construir

1. ~~Llamar a Wompi y confirmar cobro desatendido con Visa~~ — **ya no aplica**: no hay cobro
   desatendido (`PA-016`).
2. Pedir tarifas por método (PSE, Nequi, Bancolombia, tarjeta por separado) — solo publican
   la general. Sigue siendo útil para el modelo de costos.
3. Confirmar con un contador colombiano el tratamiento de IVA sobre el servicio y sobre los
   excedentes facturados.
4. Probar la API de Alegra en sandbox antes de cerrar la decisión.
5. **Fijar precios y cupos de Starter / Professional / Enterprise** sobre costo variable real
   (`PA-043`). Depende de cerrar la cotización de fuentes (`PA-040`).
6. Definir el procedimiento de conciliación de pagos que no pasan por la pasarela
   (transferencia bancaria y Efecty).

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
