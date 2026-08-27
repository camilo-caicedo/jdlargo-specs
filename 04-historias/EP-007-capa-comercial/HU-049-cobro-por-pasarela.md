---
id: HU-049
titulo: Cobro por pasarela
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-08-27
---

# HU-049 — Cobro por pasarela

## Historia

**Como** proveedor del servicio
**quiero** cobrar el plan y su excedente por los medios de pago que el cliente colombiano usa de
verdad
**para** que el dinero entre sin perseguirlo, y sin que un reintento del proveedor acabe cobrando
dos veces.

## Contexto

`ADR-0002` decidió **Wompi** por cobertura de métodos locales, por el respaldo comercial en ventas
entre empresas y porque su plan de pasarela permite negociar tarifas. Y decidió un **patrón dual**,
que no es una preferencia sino una lectura del mercado:

- **Personas y clientes pequeños:** tarjeta tokenizada, cobro automático al cierre de ciclo por
  plan más excedente.
- **Empresas:** factura al inicio del ciclo con el consumo del mes anterior y enlace de pago. Esto
  además elimina el problema del monto variable, que es donde los motores de tarjeta se rompen.

Sobre todo ello pesa `PA-016`, **bloqueante**: la documentación de Wompi indica que el protocolo
para transacciones automáticas está disponible solo para una marca de tarjetas, y los métodos de
pago locales por transferencia no son tokenizables para cobro recurrente. Si el cobro desatendido
no funciona con la marca mayoritaria, buena parte de los clientes pequeños no es cobrable de forma
automática.

La capa de cobro vive **detrás de un puerto**, no acoplada al proveedor: el sistema de pagos
inmediatos local cambiará la ecuación cuando habilite débitos automáticos.

## Criterios de aceptación

```gherkin
Escenario: Cobro automático a un cliente pequeño
  Dado una organización cliente con medio de pago tokenizado y un ciclo cerrado
  Cuando se ejecuta el cobro
  Entonces se cobra el importe del plan más el excedente del ciclo
  Y el resultado queda registrado con su referencia del proveedor
  Y el ciclo queda marcado como cobrado o como fallido, según el resultado
```

```gherkin
Escenario: Cobro a una empresa con enlace de pago
  Dado una organización cliente configurada para pago por transferencia
  Cuando cierra el ciclo
  Entonces se emite el cobro con su enlace de pago
  Y el pago se concilia cuando el proveedor lo confirma
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
Escenario: El medio de pago se guarda sin guardar la tarjeta
  Dado una organización cliente que registra su medio de pago
  Cuando se tokeniza
  Entonces se almacena únicamente la referencia del proveedor y los datos no sensibles de
  presentación
  Y ningún dato completo de tarjeta se guarda en la plataforma
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

- Dos modos de cobro, según el tipo de cliente: **tarjeta tokenizada** con cobro automático, o
  **factura más enlace de pago** (`ADR-0002`).
- Todo aviso entrante del proveedor **se verifica** —firma— y se procesa con **clave de
  idempotencia**: un reenvío no cobra ni concilia dos veces.
- La plataforma **no almacena datos completos de tarjeta**: solo la referencia del proveedor.
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
- La gestión elaborada de impagos, reintentos escalonados y recuperación de cartera.
- El débito automático por llave del sistema de pagos inmediatos local: el puerto queda listo, la
  integración llega cuando el servicio lo habilite.
- Pagos en moneda distinta del peso colombiano.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `medio_pago.organization_id` | Sí | Organización cliente existente | No |
| `medio_pago.referencia_proveedor` | Sí | Token del proveedor; **nunca datos completos de tarjeta** | Sí |
| `medio_pago.modo` | Sí | `tarjeta_tokenizada` \| `enlace_de_pago` | No |
| `cobro.ciclo_id` | Sí | Ciclo cerrado (`HU-048`) | No |
| `cobro.importe` | Sí | Plan más excedente del ciclo | No |
| `cobro.estado` | Sí | `pendiente` \| `pagado` \| `fallido` | No |
| `cobro.referencia_proveedor` | Condicional | Obligatoria una vez enviado al proveedor | No |
| `evento_pasarela.clave_idempotencia` | Sí | Única; impide procesar dos veces | No |
| `evento_pasarela.firma_verificada` | Sí | Verdadero; un aviso sin firma válida se rechaza | No |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` (Wompi, patrón dual, puerto de método de cobro),
  `08-desarrollo/arquitectura-de-aplicacion.md` (el aviso entrante se verifica, se guarda con
  clave de idempotencia y se delega)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-016` — **bloqueante y previa a construir**: si el cobro desatendido
  funciona con la marca de tarjeta mayoritaria o solo con una. Es una llamada al proveedor, no una
  decisión de arquitectura. **No pasa de `borrador` hasta que se responda.** `PA-036` — precios.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-048` (hay un importe que cobrar), `HU-046`.
- **Habilita a:** `HU-050`.
- **Riesgo:** el fallo clásico de esta integración es procesar dos veces el mismo aviso del
  proveedor. La clave de idempotencia no es una optimización: es lo que evita cobrarle dos veces a
  un cliente y descubrirlo cuando él lo reclama.
