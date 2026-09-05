---
id: HU-049
titulo: Cobro por pasarela
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-09-05
---

# HU-049 — Cobro por pasarela

> **Actualización 2026-09-05 (`PA-016`) — esta historia cambia de fondo.** El cliente descartó
> el débito automático con tarjeta: **no se le pide a nadie registrar una tarjeta**. El flujo pasa
> a ser `cierre de ciclo → factura DIAN → link de pago → conciliación`, con el medio que el
> cliente elija: **PSE, tarjeta débito/crédito, Efecty o transferencia** a la cuenta bancaria del
> negocio.
>
> Lo que **desaparece** de esta historia: tokenización de tarjetas, protocolo 3RI, 3DS para
> *payment sources*, reintentos de cobro y *dunning* automático. Wompi sigue, pero en **pago único
> por link**, que es su caso soportado sin reservas.
>
> Lo que **aparece**: conciliación de pagos que **no pasan por la pasarela** (transferencia y
> Efecty), y por tanto un proceso de cobranza con intervención humana. Ver `ADR-0002` §2.

## Historia

**Como** proveedor del servicio
**quiero** cobrar el plan y su excedente por los medios de pago que el cliente colombiano usa de
verdad
**para** que el dinero entre sin perseguirlo, y sin que un reintento del proveedor acabe cobrando
dos veces.

## Contexto

`ADR-0002` decidió **Wompi** por cobertura de métodos locales, por el respaldo comercial en ventas
entre empresas y porque su plan de pasarela permite negociar tarifas. El **patrón dual** que este
documento traía —tarjeta tokenizada para clientes pequeños, factura para empresas— quedó
**descartado** al cerrar `PA-016`.

Un solo patrón para todos, sin distinguir tipo de cliente:

1. Se cierra el ciclo y se calcula plan más excedente.
2. Se emite la **factura electrónica DIAN** (`HU-050`).
3. Se envía con un **link de pago**, y el cliente elige medio: **PSE, tarjeta débito/crédito,
   Efecty o transferencia** a la cuenta bancaria del negocio.
4. La plataforma **concilia el pago** contra la factura y libera el ciclo siguiente.

El riesgo que pesaba sobre esta historia —que el cobro desatendido solo funcionara con una marca
de tarjetas, dejando a buena parte de los clientes pequeños sin forma automática de cobro— **se
resolvió eliminando la dependencia**, no llamando al proveedor.

El costo que eso traslada: la cobranza pasa a tener intervención humana (recordatorios, mora, y
conciliación de los pagos que no cruzan la pasarela: transferencia y Efecty).

La capa de cobro vive **detrás de un puerto**, no acoplada al proveedor: el sistema de pagos
inmediatos local cambiará la ecuación cuando habilite débitos automáticos.

## Criterios de aceptación

```gherkin
Escenario: Al cerrar el ciclo se emite el cobro con su enlace de pago
  Dado una organización cliente con un ciclo cerrado
  Cuando se genera el cobro
  Entonces el importe es el del plan más el excedente del ciclo
  Y se emite un enlace de pago asociado a la factura del ciclo
  Y el enlace ofrece los medios habilitados: PSE, tarjeta, efectivo y transferencia
  Y el cobro queda en estado pendiente
```

```gherkin
Escenario: Nadie tiene que registrar una tarjeta
  Dado una organización cliente que nunca guardó un medio de pago
  Cuando se le emite un cobro
  Entonces el cobro se emite igual
  Y en ningún momento se le exige registrar ni tokenizar una tarjeta
```

```gherkin
Escenario: El pago por la pasarela se concilia solo
  Dado un cobro pendiente con enlace de pago
  Cuando el proveedor confirma el pago
  Entonces el cobro queda como pagado con su referencia del proveedor
  Y el ciclo siguiente queda liberado
```

```gherkin
Escenario: El pago que no cruza la pasarela se concilia a mano y deja rastro
  Dado un cobro pendiente que el cliente pagó por transferencia bancaria
  Cuando una persona autorizada lo concilia contra el comprobante
  Entonces el cobro queda como pagado con el medio "transferencia"
  Y queda registrado quién lo concilió, cuándo y con qué comprobante
```

```gherkin
Escenario: Un aviso repetido del proveedor no cobra ni concilia dos veces
  Dado un aviso de pago ya procesado
  Cuando el proveedor lo reenvía
  Entonces se reconoce por su clave de idempotencia
  Y no se registra un pago duplicado
  Y el reenvío queda registrado
```

```gherkin
Escenario: El aviso del proveedor se verifica antes de creerse
  Dado un aviso entrante que dice confirmar un pago
  Cuando se recibe
  Entonces se verifica su firma antes de procesarlo
  Y un aviso sin firma válida se rechaza y se registra el intento
```

```gherkin
Escenario: Un cobro fallido no interrumpe el servicio por sí solo
  Dado un cobro que falla
  Cuando se registra el fallo
  Entonces el ciclo queda como pendiente de pago
  Y se avisa al Administrador de la organización cliente
  Y ninguna suspensión del servicio ocurre de forma automática sin la política definida
```

```gherkin
Escenario: La plataforma nunca ve los datos de la tarjeta
  Dado un cliente que paga con tarjeta desde el enlace de pago
  Cuando completa el pago en la pasarela
  Entonces la plataforma almacena únicamente la referencia del proveedor y el resultado
  Y ningún dato de tarjeta pasa por la plataforma ni queda almacenado en ella
```

```gherkin
Escenario: El proveedor de cobro es reemplazable
  Dado la capa de cobro
  Cuando se consulta cómo se integra
  Entonces el proveedor está detrás de un puerto único
  Y cambiarlo no exige tocar el cierre de ciclo ni la facturación
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los cobros
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta sus cobros con su contexto de usuario propagado
  Entonces obtiene únicamente los suyos
```

## Reglas de negocio

- **Un solo modo de cobro: factura más enlace de pago** (`ADR-0002` §2). **No hay débito
  automático ni tarjetas tokenizadas**: nadie registra un medio de pago para que se le cobre solo.
- Medios habilitados en el enlace: **PSE, tarjeta débito/crédito, Efecty y transferencia** a la
  cuenta bancaria del negocio. Los dos últimos pueden no cruzar la pasarela y **se concilian a
  mano**, dejando registro de quién, cuándo y contra qué comprobante.
- Todo aviso entrante del proveedor **se verifica** —firma— y se procesa con **clave de
  idempotencia**: un reenvío no cobra ni concilia dos veces.
- La plataforma **no almacena datos de tarjeta** en ninguna forma: solo la referencia del
  proveedor y el resultado. El dato de tarjeta no pasa por ella.
- Un cobro fallido deja el ciclo pendiente de pago y genera aviso. **La suspensión del servicio no
  es automática** mientras no exista una política definida por el cliente.
- La capa de cobro vive detrás de un **puerto reemplazable**: el proveedor puede cambiar sin tocar
  el cierre de ciclo ni la facturación.
- Todo cobro queda registrado con su referencia del proveedor, su importe, su momento y su
  resultado.
- Las llaves del proveedor viven **solo en el servidor** y nunca llegan al navegador.

## Fuera de alcance

- La emisión de la factura electrónica → `HU-050`.
- El cálculo del importe → `HU-048`.
- La gestión elaborada de impagos, reintentos escalonados y recuperación de cartera. Con cobro por
  facturación esta parte es **operativa y humana**: recordatorios y seguimiento de mora, no
  *dunning* automático.
- El débito automático por llave del sistema de pagos inmediatos local: el puerto queda listo, la
  integración llega cuando el servicio lo habilite.
- Pagos en moneda distinta del peso colombiano.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `cobro.organization_id` | Sí | Organización cliente existente | No |
| `cobro.ciclo_id` | Sí | Ciclo cerrado (`HU-048`) | No |
| `cobro.factura_id` | Sí | Factura emitida (`HU-050`) | No |
| `cobro.importe` | Sí | Plan más excedente del ciclo | No |
| `cobro.enlace_pago` | Sí | Vigente y ligado a la factura; expira y se puede reemitir | No |
| `cobro.medio` | Condicional | `pse` \| `tarjeta` \| `efectivo` \| `transferencia`; obligatorio al conciliar | No |
| `cobro.estado` | Sí | `pendiente` \| `pagado` \| `vencido` | No |
| `cobro.referencia_proveedor` | Condicional | Obligatoria si el pago cruzó la pasarela | No |
| `cobro.conciliado_por` | Condicional | Persona identificada; obligatoria si el pago **no** cruzó la pasarela | No |
| `evento_pasarela.clave_idempotencia` | Sí | Única; impide procesar dos veces | No |
| `evento_pasarela.firma_verificada` | Sí | Verdadero; un aviso sin firma válida se rechaza | No |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` §2 (factura más link de pago, sin débito automático; puerto de método de cobro),
  `08-desarrollo/arquitectura-de-aplicacion.md` (el aviso entrante se verifica, se guarda con
  clave de idempotencia y se delega)

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-016` resuelta — y cambia esta historia de fondo.** No hay cobro
  desatendido: se factura y se envía link de pago. `PA-043` — precios, pendiente.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-048` (hay un importe que cobrar), `HU-046`.
- **Habilita a:** `HU-050`.
- **Riesgo:** el fallo clásico de esta integración es procesar dos veces el mismo aviso del
  proveedor. La clave de idempotencia no es una optimización: es lo que evita cobrarle dos veces a
  un cliente y descubrirlo cuando él lo reclama.
