# jdlargo-specs

Repositorio de **especificaciones** de la Plataforma JD Largo: una plataforma que **automatiza
y traza la debida diligencia** de contrapartes en materia de **SARLAFT, SAGRILAFT y PTEE**.

> El producto **no certifica**: consulta, verifica, evalúa, documenta y sugiere. La decisión y
> la responsabilidad son del cliente. Ver `ADR-0006`.

Aquí vive la definición del producto — no hay código. El backend y el frontend irán en
repos aparte, dentro de la misma carpeta contenedora `Plataforma-jdlargo`.

> **Estado: descubrimiento cerrado, listo para arrancar la Fase 0.** El cliente entregó el
> [documento funcional completo](01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md)
> el 2026-08-21 y el 2026-09-05 respondió las **39 preguntas abiertas** (`PA-001` a `PA-039`).
> El alcance se entrega **por fases verticales**: ver el [roadmap](02-producto/roadmap.md).
> Quedan seis preguntas derivadas (`PA-040` a `PA-045`); ninguna bloquea las fases 0 a 4. La
> más urgente es `PA-040`, la cotización de fuentes colombianas.

## Mapa del repo

| Carpeta | Qué contiene |
|---------|--------------|
| [`00-contexto/`](00-contexto/) | [Visión](00-contexto/vision.md) · [Actores y roles](00-contexto/actores-y-roles.md) · [Glosario](00-contexto/glosario.md) · [Normativa](00-contexto/normativa/) |
| [`01-descubrimiento/`](01-descubrimiento/) | **[Plan de definición](01-descubrimiento/plan-de-definicion.md)** · [Entregables del cliente](01-descubrimiento/entregables-cliente/) · [Preguntas abiertas](01-descubrimiento/preguntas-abiertas.md) · [Supuestos](01-descubrimiento/supuestos.md) · [Notas](01-descubrimiento/notas/) |
| [`02-producto/`](02-producto/) | [Mapa de capacidades](02-producto/mapa-de-capacidades.md) · [Flujos](02-producto/flujos/) · [Roadmap](02-producto/roadmap.md) |
| [`03-requisitos/`](03-requisitos/) | [Funcionales](03-requisitos/funcionales.md) · [No funcionales](03-requisitos/no-funcionales.md) |
| [`04-historias/`](04-historias/) | Épicas e historias de usuario · [Backlog](04-historias/backlog.md) |
| [`05-datos/`](05-datos/) | [Modelo conceptual](05-datos/modelo-conceptual.md) · [Diccionario de datos](05-datos/diccionario-de-datos.md) |
| [`06-decisiones/`](06-decisiones/) | ADRs |
| [`07-integraciones/`](07-integraciones/) | Fuentes y sistemas externos |
| [`08-desarrollo/`](08-desarrollo/) | [Arquitectura de la aplicación](08-desarrollo/arquitectura-de-aplicacion.md) · [Catálogo de librerías y entorno de IA](08-desarrollo/librerias-y-entorno-ia.md) · [Prompt para generar historias](08-desarrollo/prompts/generar-historias.md) |
| [`_plantillas/`](_plantillas/) | Plantillas de historia, épica, flujo, ADR y nota |

## Por dónde empezar

**→ [`01-descubrimiento/plan-de-definicion.md`](01-descubrimiento/plan-de-definicion.md)** —
qué hay que definir ahora, en qué orden y antes de qué fecha. Es el archivo que responde
"¿qué sigue?" en cualquier momento del proyecto.

Después:

1. [`00-contexto/vision.md`](00-contexto/vision.md) y el [glosario](00-contexto/glosario.md) para el contexto.
2. Las [preguntas abiertas](01-descubrimiento/preguntas-abiertas.md) — son la agenda del proyecto.
3. El [roadmap](02-producto/roadmap.md) y las [decisiones](06-decisiones/) para saber qué se construye y con qué.
4. Para escribir historias: [`04-historias/README.md`](04-historias/README.md) y la skill `/user-story-writing`.

Las convenciones (IDs, estados, front-matter, definition of ready) están en
[`CLAUDE.md`](CLAUDE.md).
