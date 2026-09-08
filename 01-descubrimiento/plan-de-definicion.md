---
id: DESC-plan
estado: vivo
actualizado: 2026-09-05
---

# Plan de definición — qué hacer ahora

> **Este es el archivo que se abre primero.** Responde "¿qué sigue?" en cualquier momento
> del proyecto. Si algo aquí queda desactualizado, se corrige aquí mismo.

## La regla

**La definición va un bloque por delante de la construcción.** Nunca se escribe una historia
de un bloque cuyo contexto no esté aprobado, y nunca se empieza a construir un bloque cuyas
preguntas bloqueantes sigan abiertas.

## Dónde mirar cada cosa

| Pregunta | Dónde |
|---|---|
| ¿Qué sigue y en qué orden? | **Este archivo** |
| ¿Qué falta por saber? | [`preguntas-abiertas.md`](preguntas-abiertas.md) |
| ¿Qué asumimos mientras tanto? | [`supuestos.md`](supuestos.md) |
| ¿Cuándo se necesita cada cosa? | [`../02-producto/roadmap.md`](../02-producto/roadmap.md) |
| ¿Qué ya está decidido y por qué? | [`../06-decisiones/`](../06-decisiones/) |
| ¿Con qué se construye? | [`../08-desarrollo/librerias-y-entorno-ia.md`](../08-desarrollo/librerias-y-entorno-ia.md) |

**Herramientas:** `/user-story-writing` para historias · `/engineering:architecture` para
nuevos ADR · `/engineering:code-review` obligatorio en RLS, cuotas y cobro.

> **Actualización 2026-08-25.** Juan David entregó su
> [documento funcional v2](entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md),
> que cerró ocho preguntas abiertas y abrió siete nuevas, entre ellas `PA-023`.
>
> **Actualización 2026-09-05.** Juan David respondió **las 39 preguntas abiertas**
> (`PA-001` a `PA-039`). Ya no queda ninguna pregunta bloqueante para construir. Lo que quedó
> son seis preguntas derivadas (`PA-040` a `PA-045`), ninguna de las cuales frena las fases 0
> a 4, y cinco ADR nuevos (`ADR-0006` a `ADR-0010`).

## Dónde estamos hoy (05-sep-2026)

**Decidido:** el stack (`ADR-0001`), el cobro por facturación (`ADR-0002`), la configuración
como datos (`ADR-0004`), la procedencia del dato (`ADR-0005`), el posicionamiento sin
certificación (`ADR-0006`), la retención y la bitácora (`ADR-0007`), el matching (`ADR-0008`),
el monitoreo continuo (`ADR-0009`) y la firma (`ADR-0010`).

**Definido con las respuestas de Juan David:** `vision.md`, `actores-y-roles.md`, `glosario.md`,
`no-funcionales.md`, el catálogo de fuentes y el modelo comercial.

**Sin definir:** `funcionales.md` (los `RF-xxx` siguen en `TBD`), los flujos `FL-xxx`, y el
alcance del MVP4 (salida por API hacia otros sistemas del cliente, `PA-010`).

La brecha ya no es el "qué": es que el "qué" está en historias y en ADR, pero todavía no en
requisitos funcionales numerados ni en flujos dibujados.

## Qué debe estar aprobado antes de cada bloque

| Bloque | Definir antes de | Artefactos que deben estar aprobados | Preguntas que deben estar cerradas |
|---|---|---|---|
| **0 · Cimientos** | **Ya** | `vision.md`, `EP-000`. Contexto y modelo ya están listos | Ninguna — no está bloqueada |
| **1 · Expediente a mano** | Semana 4 | `FL-001` (recorrido completo de vinculación), plantillas base y matriz de requisitos del cliente ancla, `EP-001` | Ninguna. `PA-018` y `PA-042` cerradas (2026-09-07) |
| **2 · Extracción** | Semana 9 | Catálogo de tipos de documento y campos a extraer, `EP-002` | `PA-045` (proveedor de IA y su contrato) |
| **3 · Verificación** | Semana 14 | `FL-002` (screening y gestión de alertas), `EP-003` | **`PA-040`** — sin cotización ni vía de conexión no se puede construir la integración |
| **4 · Riesgo** | Semana 19 | Metodología del cliente ancla, reglas de relaciones, `EP-004` | Ninguna. `PA-006` y `PA-034` cerradas |
| **5 · Autogestión** | Semana 25 | Diseño de la interfaz de configuración, `EP-005` | Ninguna. `PA-017` cerrada: plantillas base + ajuste del cliente |
| **6 · Monitoreo** | Semana 30 | `FL-003` (eventos y renovaciones), `EP-006` | Ninguna. `PA-011`, `PA-014` y `PA-035` cerradas → `ADR-0009` |
| **C · Comercial** | 2 semanas antes de arrancarla | Planes, cupos y precios; `FL-004`; `EP-00C` | **`PA-043`** (precios), que depende de `PA-040` |

La definición va siempre una fase por delante de la construcción. La capa comercial es móvil:
se ubica donde aparezca el primer cliente que pague.

## Las tres sesiones con Juan David que hay que agendar

| Sesión | Cuándo | Agenda | Preguntas |
|---|---|---|---|
| ~~1 · Producto y actores~~ | — | — | **Hecha.** `PA-001` a `PA-004`, `PA-007`, `PA-013` cerradas |
| ~~2 · Riesgo y evidencia~~ | — | — | **Hecha.** `PA-006`, `PA-009` cerradas |
| ~~3 · Comercial~~ | — | — | **Hecha.** `PA-011`, `PA-014` cerradas |
| ~~4 · Recorrido de la contraparte~~ | — | — | **`PA-042` resuelta sin sesión, por Camilo (2026-09-07):** formulario manual en Fase 1 (`HU-012`), autollenado en `EP-002` — ya construido (`HU-010`, portal por enlace). El detalle de cómo se confirma un campo autollenado queda en `PA-050`, para cuando arranque `EP-002` |
| **5 · Precios** | Antes del bloque C, después de `PA-040` | Precio y cupo de cada plan sobre costo variable real | `PA-043`, `PA-044` |

Cada sesión deja una nota en [`notas/`](notas/) con la plantilla
[`nota-reunion.md`](../_plantillas/nota-reunion.md).

## La llamada que no depende de Juan David

De las dos que había, una se cerró sola: **`PA-016`** dejó de importar porque Juan David
descartó el débito automático con tarjeta y optó por facturación (`ADR-0002`). Queda una, y
sigue siendo la única gestión capaz de tumbar el modelo de negocio:

1. **`PA-040`** — establecer con qué fuentes colombianas se puede conectar directamente y a
   qué costo real, y cotizar formalmente la alternativa intermediada (Tusdatos.co, Datacrédito
   Experian, Compliance.com.co). Juan David estima $1.000–$2.000 COP por consulta, pero es una
   estimación, no una cotización (`SUP-009`, `SUP-010`). Define el modelo de precios entero.

**Esta semana.** Y en paralelo, Juan David tiene pendiente `PA-041` (firma digital y validez
del OTP por correo), que no bloquea el MVP pero sí condiciona `ADR-0010`.

## Tu próxima tarea concreta

~~Llenar `00-contexto/`~~ — el documento de Juan David ya permitió redactar
[`glosario.md`](../00-contexto/glosario.md) y [`actores-y-roles.md`](../00-contexto/actores-y-roles.md).

Lo que queda, en orden:

~~Resolver `PA-023`~~ — resuelta: el alcance se mantiene y se entrega **por fases verticales**
(ver [roadmap](../02-producto/roadmap.md)).

1. **Validar el faseo con Juan David**, en particular que la Fase 1 —sin IA ni screening— le
   sirva para usarla de verdad. Todo el plan depende de eso. Añadir a esa conversación el
   encaje con su escalera MVP1–MVP5 y dónde entra el **MVP4** (salida por API, `PA-010`).
2. ~~`vision.md`~~ — **hecha** con las respuestas a `PA-001`, `PA-003`, `PA-004`, `PA-010`,
   `PA-022` y `PA-023`.
3. ~~`EP-000` y `EP-001`~~ — **hechas**. La Fase 0 no depende de ninguna pregunta abierta: se
   puede arrancar ya.
4. **`03-requisitos/funcionales.md`** — es el hueco más grande que queda. Los `RF-xxx` siguen
   en `TBD` mientras las historias ya están escritas; la trazabilidad está rota por ahí.
5. **`FL-001`** — el recorrido completo de vinculación. `PA-042`, que lo condicionaba, ya está
   cerrada (2026-09-07).

## El ciclo de trabajo, en cinco pasos

1. Sesión con Juan David → nota en `01-descubrimiento/notas/`.
2. Lo que quedó en firme sube a `00-contexto/` o `02-producto/`; lo que quedó en el aire baja
   como `PA-xxx` nueva.
3. Capacidad estable → se abre una épica en `04-historias/`.
4. Épica → historias con `/user-story-writing`.
5. Decisión con alternativas y consecuencias → ADR con `/engineering:architecture`.
