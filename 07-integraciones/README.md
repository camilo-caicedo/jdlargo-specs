---
id: INT-catalogo
estado: borrador
actualizado: 2026-08-21
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

| ID | Sistema / fuente | Para qué | Tipo | Estado |
|----|------------------|----------|------|--------|
| INT-001 | **OpenSanctions** | Índice consolidado internacional: OFAC, ONU, UE, PEPs y cientos de fuentes normalizadas | API SaaS | Elegido (`ADR-0001`) |
| INT-002 | **Proveedor local colombiano** (por definir) | Procuraduría, Contraloría, Policía, RUES | API | **Por cotizar** (`PA-012`) |
| INT-003 | Registro mercantil / RUES | Datos societarios, representante legal, beneficiario final | Por definir | Por evaluar |

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

### INT-002 — Fuentes colombianas

OpenSanctions **no cubre** Procuraduría, Contraloría, Policía ni RUES, y esas son buena
parte de lo que el cliente colombiano entiende por "la consulta". No tienen API pública
utilizable, así que hay que comprarlas.

Por cotizar (`PA-012`), en orden de prioridad:

- [Tusdatos.co](https://www.tusdatos.co/pages/cumplimiento)
- [Datacrédito Experian](https://www.datacredito.com.co/empresas/listas-restrictivas)
- [Compliance.com.co](https://www.compliance.com.co/)

**Su precio por consulta define el modelo de precios del producto, no al revés.** Es la
cotización más urgente del proyecto.

## El modelo de costos, y su trampa

Con un presupuesto pre-ingresos de 100 USD/mes y 0,10 € por llamada, el techo son unas
**800-900 consultas mensuales**. Suficiente para un piloto de consulta puntual.

El riesgo aparece con el **monitoreo continuo** (`PA-011`): un solo cliente con 5.000
contrapartes re-escaneadas cada mes son 5.000 llamadas, unos 500 €. Con una suscripción de
tarifa plana, ese cliente deja el negocio en pérdida y no hay forma de enterarse hasta que
llega la factura.

Consecuencias de diseño, desde el día uno:

1. **Medir el consumo por organización** en la misma tabla inmutable que sirve de evidencia
   de auditoría. Es el instrumento que responderá `PA-011` y `PA-014` con datos reales en
   tres meses, en vez de adivinarlas hoy.
2. **Deduplicar por identidad**: no re-consultar la misma contraparte dos veces en una ventana.
3. **Re-screening incremental**: cuando el monitoreo exista, re-evaluar contra lo que cambió
   en el dataset, no lanzar una llamada por contraparte. Este requisito es, por sí solo, el
   principal argumento a favor del auto-hospedaje.
4. **Cuotas duras por plan**, aplicadas antes de gastar, no después.

## Reglas para toda integración

Por cada una, documentar: qué dato entra y sale, frecuencia, límites, **costo por llamada**,
qué ocurre si el servicio no responde, y **cómo se evidencia la consulta para auditoría**.

Regla dura: **de cada consulta se guarda el snapshot del resultado, nunca un enlace.** Un
enlace cambia mañana; la evidencia ante un supervisor tiene que ser reconstruible tal como
se vio el día de la decisión.
