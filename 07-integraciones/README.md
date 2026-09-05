---
id: INT-catalogo
estado: propuesto
actualizado: 2026-09-05
---

# Integraciones externas

## El punto de partida

El valor del producto **no está en el CRUD sino en el dato consolidado y en la calidad del
cruce**. Conviene desarmar una idea antes de seguir: nadie consulta "1.800 fuentes en
tiempo real" cuando el usuario aprieta un botón. Lo que existe en todos los productos de
este mercado es un **índice consolidado**, actualizado de forma continua por detrás, contra
el que la consulta corre en milisegundos.

Construir y mantener esa ingesta es un producto completo en sí mismo. **Decisión: se compra
el dato consolidado y se compite en la capa de arriba** — workflow, evidencia, justificación
y reportes. Ver `ADR-0001`.

## Catálogo

El cliente fijó el catálogo inicial al cerrar `PA-005`. Se agrupa por naturaleza, porque la
forma de conectarse y el costo cambian por completo entre un grupo y otro.

| ID | Sistema / fuente | Para qué | Tipo | Estado |
|----|------------------|----------|------|--------|
| INT-001 | **OpenSanctions** | Índice consolidado internacional: OFAC-SDN, ONU (Res. 1267), UE, PEPs y cientos de fuentes normalizadas | API SaaS | Elegido (`ADR-0001`) |
| INT-002 | **Antecedentes penales** — Policía Nacional | Antecedentes de la persona natural | Directa o proveedor | **Por definir vía** (`PA-040`) |
| INT-003 | **Antecedentes disciplinarios** — Procuraduría | Inhabilidades y sanciones disciplinarias | Directa o proveedor | **Por definir vía** (`PA-040`) |
| INT-004 | **Antecedentes fiscales** — Contraloría | Responsabilidad fiscal | Directa o proveedor | **Por definir vía** (`PA-040`) |
| INT-005 | **Listas UIAF Colombia** | Listado nacional | Directa o proveedor | **Por definir vía** (`PA-040`) |
| INT-006 | **GAFI / FATF** | Jurisdicciones de alto riesgo y bajo monitoreo | Publicación periódica | Por evaluar |
| INT-007 | **RUNT / SIMIT** | Antecedentes de tránsito — clave para el sector transporte (conductor, propietario, poseedor) | Directa o proveedor | **Por definir vía** (`PA-040`) |
| INT-008 | **Registro mercantil / RUES** | Datos societarios, representante legal, insumo de beneficiario final | Por definir | Por evaluar |

> ONU y OFAC-SDN entran por INT-001 (OpenSanctions ya los consolida y normaliza). Consultarlos
> por separado sería pagar dos veces por el mismo dato.

### Cómo se conecta: directa antes que intermediada

Decisión del cliente (`PA-005`): **la conexión ideal es directa entre la plataforma y cada
entidad**; el proveedor intermediario es el plan B, no el punto de partida. Qué fuentes
admiten conexión directa y con qué condiciones es lo que falta por establecer (`PA-040`), y es
la investigación más urgente junto con la cotización.

### Qué se guarda de cada consulta

De **toda** consulta, sea directa o por proveedor, se persiste:

`fuente · proveedor · fecha y hora · versión o corte del dato · request/response o evidencia
equivalente · resultado · costo`

El costo se guarda incluso cuando la consulta es gratuita: es la única forma de saber después
qué habría costado, y de comparar la vía directa contra la intermediada con datos y no con
impresiones.

### INT-001 — OpenSanctions

Dos modalidades, y la elección depende del volumen:

| | API SaaS | Auto-hospedado (`yente`) |
|---|---|---|
| Costo | **0,10 € por llamada exitosa**; precios por volumen desde 20.000/mes | Tarifa plana de licencia de datos, uso interno ilimitado |
| Infraestructura | Ninguna | Elasticsearch u OpenSearch, memoria y almacenamiento rápido |
| Puesta en marcha | Minutos | **De días a semanas** de ingeniería |
| Mantenimiento | Ninguno | Vigilar reindexaciones (varias al día) y actualizar `yente` |
| Uso gratuito | Solo para uso **no comercial** | — |

**Decisión: API SaaS.** Auto-hospedar es correcto dentro de un año con volumen real; hoy se
comería el piloto entero. Como lo plantean ellos mismos: *"la licencia es solo el precio de
la casa, no el costo de vivir en ella"*.

Se reevalúa (→ `ADR-0003`) cuando se active el monitoreo continuo o cuando el volumen
mensual supere el punto de equilibrio.

- Docs: <https://www.opensanctions.org/docs/api/>
- Costo SaaS vs on-premise: <https://www.opensanctions.org/faq/api/license-cost/>

### INT-002 a INT-007 — Fuentes colombianas

OpenSanctions **no cubre** Procuraduría, Contraloría, Policía, RUNT/SIMIT ni RUES, y esas son
buena parte de lo que el cliente colombiano entiende por "la consulta".

**Costo de referencia: $1.000–$2.000 COP por consulta** (`PA-012`), según el cliente, para
consulta hecha directamente desde la plataforma sin intermediación de terceros.

> ⚠️ **Ese número no es un costo definitivo y no debe usarse para fijar precios.** Es un
> costo provisional para el modelo financiero. Antes de publicar un precio hay que cotizar
> formalmente y negociar API, volumen, SLA y licenciamiento. Y hay que presupuestar por
> **"consulta de persona/entidad + paquete de fuentes"**, no fuente por fuente: es como se
> comporta el gasto real y como lo cobran los proveedores.

Candidatos si se va por la vía intermediada, en orden de prioridad:

- [Tusdatos.co](https://www.tusdatos.co/pages/cumplimiento)
- [Datacrédito Experian](https://www.datacredito.com.co/empresas/listas-restrictivas)
- [Compliance.com.co](https://www.compliance.com.co/)

**El precio por consulta define el modelo de precios del producto, no al revés** (`PA-043`).

## El modelo de costos, y su trampa

Con un presupuesto pre-ingresos de 100 USD/mes y 0,10 € por llamada, el techo son unas
**800-900 consultas mensuales**. Suficiente para un piloto de consulta puntual.

El riesgo se materializó: el **monitoreo continuo está confirmado en alcance** (`PA-011`) y
la capacidad objetivo del MVP es de **10.000 a 50.000 contrapartes por organización cliente**
(`PA-014`). Un cliente con 50.000 contrapartes re-escaneadas cada mes son 50.000 llamadas. Con
una suscripción de tarifa plana, ese cliente deja el negocio en pérdida y no hay forma de
enterarse hasta que llega la factura.

Referencia real que dio el cliente: una empresa de transporte de carga hace **~1.000 consultas
al mes** (`PA-014`). Es el dato con el que se dimensiona el primer plan — no la cifra con la
que se dimensiona la arquitectura.

Consecuencias de diseño, desde el día uno:

1. **Medir el consumo por organización** en la misma tabla inmutable que sirve de evidencia
   de auditoría. Es el instrumento que permitirá fijar los precios de `PA-043` con datos
   reales en tres meses, en vez de adivinarlos hoy.
2. **Deduplicar por identidad**: no re-consultar la misma contraparte dos veces en una ventana.
3. **Re-screening incremental**: re-evaluar contra lo que cambió en el conjunto de datos, no
   lanzar una llamada por contraparte. Este requisito es, por sí solo, el principal argumento
   a favor del auto-hospedaje.
4. **Cuotas por plan aplicadas antes de gastar**, no después — con una excepción: un
   monitoreo en curso **no se corta en silencio** por un límite. Se avisa al 80 %, 90 % y
   100 %, y se ofrece excedente o cambio de plan (`ADR-0002` §2b, `ADR-0009`).
5. **Planificador por riesgo y por evento**, no barrido global: alto mensual o trimestral,
   medio semestral, bajo anual, como configuración de referencia (`ADR-0009`, `PA-035`).

## Reglas para toda integración

Por cada una, documentar: qué dato entra y sale, frecuencia, límites, **costo por llamada**,
qué ocurre si el servicio no responde, y **cómo se evidencia la consulta para auditoría**.

Regla dura: **de cada consulta se guarda el snapshot del resultado, nunca un enlace.** Un
enlace cambia mañana; la evidencia ante un supervisor tiene que ser reconstruible tal como
se vio el día de la decisión.
