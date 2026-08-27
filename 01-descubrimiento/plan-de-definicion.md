---
id: DESC-plan
estado: vivo
actualizado: 2026-08-21
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

> **Actualización 2026-08-25.** El cliente entregó su
> [documento funcional v2](entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md),
> que cierra ocho preguntas abiertas y abre siete nuevas. La más importante es **`PA-023`**:
> el alcance descrito excede el calendario planificado. **Esa decisión bloquea el arranque** —
> ver [roadmap](../02-producto/roadmap.md).

## Dónde estamos hoy (21-ago-2026)

**Decidido:** el stack (`ADR-0001`), el cobro y la facturación (`ADR-0002`), las
integraciones de screening, y el plan de 20 semanas.

**Sin definir:** casi todo `00-contexto/`. La visión, los actores y el glosario siguen en `TBD`.

Esa es la brecha exacta del proyecto en este momento: **el "cómo" está decidido y el "qué"
está en blanco.** Toca cerrarla antes de la semana 1, porque no se pueden escribir historias
sobre roles y organizaciones sin saber quiénes son los roles.

## Qué debe estar aprobado antes de cada bloque

| Bloque | Definir antes de | Artefactos que deben estar aprobados | Preguntas que deben estar cerradas |
|---|---|---|---|
| **0 · Cimientos** | **Ya** | `vision.md`, `EP-000`. Contexto y modelo ya están listos | Ninguna — no está bloqueada |
| **1 · Expediente a mano** | Semana 4 | `FL-001` (recorrido completo de vinculación), matriz de requisitos del cliente ancla, `EP-001` | `PA-018` |
| **2 · Extracción** | Semana 9 | Catálogo de tipos de documento y campos a extraer, `EP-002` | `PA-021` |
| **3 · Verificación** | Semana 14 | `FL-002` (screening y gestión de alertas), `EP-003` | `PA-012` |
| **4 · Riesgo** | Semana 19 | Metodología del cliente ancla, reglas de relaciones, `EP-004` | `PA-006` (cerrada) |
| **5 · Autogestión** | Semana 25 | Diseño de la interfaz de configuración, `EP-005` | `PA-017` |
| **6 · Monitoreo** | Semana 30 | `FL-003` (eventos y renovaciones), `EP-006` | `PA-011` (cerrada), `PA-014` |
| **C · Comercial** | 2 semanas antes de arrancarla | Planes, cupos y precios; `FL-004`; `EP-00C` | `PA-012`, `PA-016` |

La definición va siempre una fase por delante de la construcción. La capa comercial es móvil:
se ubica donde aparezca el primer cliente que pague.

## Las tres sesiones con el cliente que hay que agendar

| Sesión | Cuándo | Agenda | Preguntas |
|---|---|---|---|
| **1 · Producto y actores** | Esta semana | Qué es certificar, a quién se evalúa, quién usa la plataforma y con qué rol, qué se lleva el usuario al final | `PA-001` a `PA-004`, `PA-007`, `PA-013` |
| **2 · Riesgo y evidencia** | En 2 semanas | Qué dispara una alerta, cómo se califica el riesgo, qué evidencia exige un supervisor | `PA-006`, `PA-009` |
| **3 · Comercial** | En 4 semanas | Planes, cupos, monitoreo continuo, volumen esperado por cliente | `PA-011`, `PA-014` |

Cada sesión deja una nota en [`notas/`](notas/) con la plantilla
[`nota-reunion.md`](../_plantillas/nota-reunion.md).

## Las dos llamadas que no dependen del cliente

Estas no son preguntas para Juan David y **son las únicas que pueden tumbar el plan**:

1. **`PA-012`** — cotizar al proveedor local de fuentes colombianas (Tusdatos.co,
   Datacrédito Experian, Compliance.com.co). Define el modelo de precios.
2. **`PA-016`** — confirmar con Wompi si el cobro desatendido funciona con Visa o solo con
   Mastercard. Define el patrón de cobro del bloque 4.

**Ambas, esta semana.**

## Tu próxima tarea concreta

~~Llenar `00-contexto/`~~ — el documento del cliente ya permitió redactar
[`glosario.md`](../00-contexto/glosario.md) y [`actores-y-roles.md`](../00-contexto/actores-y-roles.md).

Lo que queda, en orden:

~~Resolver `PA-023`~~ — resuelta: el alcance se mantiene y se entrega **por fases verticales**
(ver [roadmap](../02-producto/roadmap.md)).

1. **Validar el faseo con Juan David**, en particular que la Fase 1 —sin IA ni screening— le
   sirva para usarla de verdad. Todo el plan depende de eso.
2. **`vision.md`**, con el posicionamiento de la §39 y los **no-objetivos**, que la §34 del
   documento del cliente prácticamente redacta.
3. **`EP-000` (cimientos) y `EP-001` (un expediente completo, a mano)** con `/user-story-writing`.
   La Fase 0 no depende de ninguna pregunta abierta: se puede arrancar ya.

## El ciclo de trabajo, en cinco pasos

1. Sesión con el cliente → nota en `01-descubrimiento/notas/`.
2. Lo que quedó en firme sube a `00-contexto/` o `02-producto/`; lo que quedó en el aire baja
   como `PA-xxx` nueva.
3. Capacidad estable → se abre una épica en `04-historias/`.
4. Épica → historias con `/user-story-writing`.
5. Decisión con alternativas y consecuencias → ADR con `/engineering:architecture`.
