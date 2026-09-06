---
id: EP-008
titulo: Salida de datos hacia otros sistemas
estado: borrador
capacidad: CAP-08
actualizado: 2026-09-06
---

# EP-008 — Salida de datos hacia otros sistemas

## Objetivo

Que la información de un expediente de debida diligencia salga de la plataforma hacia los
sistemas del cliente (ERP, TMS u otros) que la necesitan para sus propios procesos, sin que el
cliente tenga que copiarla a mano.

## Por qué existe

Cierra `PA-010`. La plataforma **no reemplaza** ningún sistema del cliente: los alimenta. El
cliente confirmó que le interesa una conexión que extraiga la información hacia otro sistema,
por dos vías posibles que no son excluyentes:

1. Una conexión por interfaz de programación.
2. Descarga (o entrega) de un archivo estructurado (JSON o TXT) configurable, que el cliente
   carga en su otro sistema.

En la escalera de MVPs del cliente (`02-producto/roadmap.md`) esto es el **MVP4**, y encaja
después de la Fase 4, antes o en paralelo con la Fase 6.

## Alcance

**Incluye:**

- Definir, por organización cliente, qué campos del expediente son exportables.
- Salida por archivo estructurado (JSON/TXT) configurable.
- Salida por interfaz de programación para que el sistema del cliente consulte o reciba la
  información.
- Autenticación de la integración y bitácora de cada extracción (quién, qué organización, qué
  expediente, cuándo, qué campos).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Migración de contrapartes y expedientes vigentes de un sistema anterior del cliente | Fuera de esta épica | `PA-010` lo trata como caso de migración de entrada (plantillas controladas), no como salida de datos; es la dirección opuesta a lo que cubre `CAP-08` |
| Conector específico a un ERP o TMS puntual | Después del MVP | No hay todavía un sistema destino identificado; se prioriza una salida genérica (API + archivo) sobre un conector a la medida |
| Sincronización en tiempo real bidireccional | TBD | Depende de `PA-046`; no se da por hecho hasta que se responda |

## Actores involucrados

- **Cliente del SaaS** — decide qué sistema propio recibe la información y con qué frecuencia.
- **Administrador** — configura qué se exporta y por qué medio.
- **Sistema** — genera el archivo o atiende la interfaz de programación; nunca decide qué se
  entrega, solo ejecuta lo configurado.
- **Sistema externo del cliente** — consume la salida (no es un actor de la plataforma; es
  quien recibe los datos).

## Criterios de éxito

1. Un expediente cerrado puede salir de la plataforma sin copiar y pegar datos a mano.
2. Cada extracción queda registrada en la bitácora: quién la pidió (usuario o integración),
   qué organización, qué expediente, qué campos y cuándo.
3. Ninguna organización puede exportar datos de otra (aislamiento multi-tenant, igual que en el
   resto del producto).
4. El formato de salida (JSON/TXT) es configurable por organización, no fijo en el código.

## Historias

> Pendientes de escribir con `/user-story-writing`. Se listan como candidatas para reservar
> numeración, no como historias ya redactadas.

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-051` | Configurar qué campos del expediente son exportables por organización | Must | pendiente |
| `HU-052` | Exportar un expediente cerrado a archivo estructurado (JSON/TXT) | Must | pendiente |
| `HU-053` | Exponer una interfaz de programación para que el sistema del cliente extraiga expedientes | Should | pendiente |
| `HU-054` | Autenticar la integración y dejar bitácora de cada extracción | Must | pendiente |

## Dependencias

- **Preguntas abiertas:**
  - `PA-010` (resuelta) — origen de esta épica.
  - **`PA-046`** (nueva) — ¿qué sistema(s) destino tiene hoy el cliente, y por cuál medio
    concreto prefiere recibir la información: interfaz de programación que exponemos nosotros,
    archivo por descarga, o entrega por SFTP/correo?
  - **`PA-047`** (nueva) — ¿la salida se dispara bajo demanda, de forma programada, o por
    evento (p. ej. al cerrar un expediente)? Define si esta épica necesita webhooks o solo
    exportación por lote.
  - **`PA-048`** (nueva) — ¿hay campos del expediente que no pueden salir hacia el sistema del
    cliente por tratarse de datos personales sensibles o de terceros (Ley 1581 de 2012)?
- **Otras épicas:** `EP-000` (bitácora y aislamiento multi-tenant), `EP-001` (expediente
  cerrado, que es lo que esta épica exporta).

## Riesgo abierto

Esta épica **no tiene todavía ni una historia redactada ni un RF numerado**
(`03-requisitos/funcionales.md`, nota sobre `CAP-08`). Antes de escribir las historias con
`/user-story-writing` hace falta cerrar al menos `PA-046`, porque cambia si la épica construye
una interfaz de programación, un exportador de archivos, o ambos desde el inicio.
