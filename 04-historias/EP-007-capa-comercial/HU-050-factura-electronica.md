---
id: HU-050
titulo: Factura electrónica
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-08-27
---

# HU-050 — Factura electrónica

## Historia

**Como** Cliente del SaaS
**quiero** recibir una factura electrónica válida por lo que pago
**para** poder deducir el gasto, que es la razón por la que mi área contable la exige.

## Contexto

`ADR-0002` lo clasifica sin rodeos: **no es un trámite, es un requisito comercial**. Un cliente
empresarial colombiano necesita la factura electrónica para deducir el gasto, y sin ella el
producto no se puede vender a empresas.

La decisión tomada es **Alegra para el piloto**, con la interfaz de programación de Factus como
alternativa a revisar. La emisión ocurre por interfaz, no a mano.

Hay una consecuencia de diseño que conviene fijar: la factura se emite **a partir del ciclo
cerrado** (`HU-048`), no de una cifra que alguien escriba. Y el detalle de la factura tiene que
poder rastrearse hasta las consultas que lo componen, porque un cliente que ve un cargo por
excedente va a preguntar.

`ADR-0002` deja además un pendiente que no es del producto sino previo: confirmar con un contador
el tratamiento del impuesto sobre el servicio y sobre los excedentes facturados.

## Criterios de aceptación

```gherkin
Escenario: Emitir la factura de un ciclo cerrado
  Dado un ciclo cerrado con su importe calculado
  Cuando se emite la factura
  Entonces se genera a partir de los datos del ciclo, no de una cifra escrita a mano
  Y queda registrada con su número, su fecha y su referencia ante la autoridad tributaria
  Y el cliente la recibe
```

```gherkin
Escenario: El detalle de la factura se puede rastrear
  Dado una factura con un cargo por excedente
  Cuando se consulta su detalle
  Entonces se obtienen las consultas que componen ese excedente, con su fecha y su fuente
  Y la suma coincide con el importe facturado
```

```gherkin
Escenario: Una factura emitida no se modifica
  Dado una factura ya emitida
  Cuando se intenta modificarla
  Entonces la operación es rechazada
  Y cualquier corrección se hace con el documento que corresponda, no editando la original
```

```gherkin
Escenario: Fallo del proveedor de facturación
  Dado un proveedor de facturación que no responde
  Cuando se intenta emitir
  Entonces la emisión queda pendiente y reintentable
  Y no se genera un número de factura consumido en falso
  Y el fallo queda registrado
```

```gherkin
Escenario: Un reintento no emite dos facturas
  Dado una emisión que se reintenta
  Cuando se ejecuta el reintento con la misma clave de idempotencia
  Entonces no se emite una segunda factura por el mismo ciclo
```

```gherkin
Escenario: Los datos de facturación del cliente son suyos y verificables
  Dado una organización cliente sin datos de facturación completos
  Cuando llega el momento de emitir
  Entonces la emisión se detiene indicando qué datos faltan
  Y se avisa al Administrador de esa organización cliente
```

```gherkin
Escenario: El proveedor de facturación es reemplazable
  Dado la capa de facturación
  Cuando se consulta cómo se integra
  Entonces el proveedor está detrás de un puerto único
  Y cambiarlo no exige tocar el cierre de ciclo ni el cobro
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las facturas
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta sus facturas con su contexto de usuario propagado
  Entonces obtiene únicamente las suyas
```

## Reglas de negocio

- La factura se emite **desde el ciclo cerrado**, nunca desde una cifra introducida a mano.
- Una factura emitida es **inmutable**. Corregir se hace con el documento que la normativa prevea,
  no editando la original.
- El detalle debe poder **rastrearse hasta las consultas** que lo componen.
- La emisión es **idempotente y reintentable**: un fallo del proveedor no consume numeración ni
  produce facturas duplicadas.
- Sin datos de facturación completos del cliente **no se emite**, y se avisa.
- El proveedor de facturación vive detrás de un **puerto reemplazable** (Alegra en el piloto,
  Factus como alternativa).
- El tratamiento del impuesto sobre el servicio y sobre los excedentes se aplica según lo que
  confirme la asesoría contable. **Está pendiente en `ADR-0002` y no se supone aquí.**

## Fuera de alcance

- El cobro del importe → `HU-049`.
- La contabilidad del negocio: la cubre el proveedor de facturación si hace falta, no el producto.
- Notas de crédito y ajustes complejos: se emiten por el proveedor cuando el caso aparezca.
- Facturación a clientes fuera de Colombia.
- La determinación del tratamiento tributario: es del contador, no del producto.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `factura.organization_id` | Sí | Organización cliente existente | No |
| `factura.ciclo_id` | Sí | Ciclo cerrado (`HU-048`); uno por ciclo | No |
| `factura.numero` | Sí | Asignado por el proveedor de facturación | No |
| `factura.fecha_emision` | Sí | Momento; se escribe una sola vez | No |
| `factura.importe` | Sí | Coincide con el importe del ciclo | No |
| `factura.detalle` | Sí | Concepto de plan y concepto de excedente, rastreables | No |
| `factura.referencia_autoridad` | Sí | Identificación del documento ante la autoridad tributaria | No |
| `factura.estado` | Sí | `pendiente` \| `emitida` \| `fallida` | No |
| `factura.clave_idempotencia` | Sí | Única por ciclo; impide emitir dos veces | No |
| `datos_facturacion.organization_id` | Sí | Razón social, identificación tributaria, dirección y correo | Sí |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` (facturación electrónica por interfaz de programación; Alegra en el
  piloto; puerto reemplazable)
- Normativa: facturación electrónica obligatoria en Colombia `(por validar contra fuente oficial
  y con asesoría contable, ver ADR-0002 y SUP-008)`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-036` (precios e importes), `PA-038` (si el alta es por autoservicio,
  los datos de facturación hay que pedirlos en el registro). **Queda en `borrador`.**
- **Supuestos:** `SUP-008` (las referencias normativas del proyecto no están verificadas; aquí
  además hay un pendiente explícito de asesoría contable).
- **Depende de:** `HU-048` (ciclo cerrado), `HU-049` (el cobro que la acompaña).
- **Habilita a:** vender a clientes empresariales, que sin factura electrónica no compran.
- **Riesgo:** consumir numeración de facturación en un intento fallido. Es un problema
  administrativo pequeño que se vuelve incómodo de explicar ante la autoridad tributaria, y se
  evita con idempotencia desde el primer día.
