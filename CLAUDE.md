# jdlargo-specs — reglas del repositorio

Repo de **definición** del proyecto Plataforma JD Largo (SARLAFT / SAGRILAFT / PTEE).
El contexto de producto y el mapa de repos están en `../CLAUDE.md`; aquí van solo las
reglas de cómo se escribe y se organiza la documentación.

Aquí **no hay código**: solo Markdown y diagramas.

## Estructura

```
00-contexto/          Qué es el producto, para quién y en qué marco normativo
  vision.md           Problema, propuesta de valor, objetivos y no-objetivos
  actores-y-roles.md  Actores del sistema y permisos por rol
  glosario.md         Lenguaje ubicuo del dominio (fuente de verdad terminológica)
  normativa/          Un archivo por marco: sarlaft, sagrilaft, ptee
01-descubrimiento/    Lo que todavía no sabemos
  preguntas-abiertas.md  PA-xxx pendientes de respuesta del cliente
  supuestos.md           SUP-xxx que asumimos mientras no haya respuesta
  notas/                 Notas de reuniones y entrevistas
02-producto/          Qué debe hacer
  mapa-de-capacidades.md  Capacidades → épicas (EP-xxx)
  flujos/                 FL-xxx: flujos end-to-end
  roadmap.md              Orden y fases de entrega
03-requisitos/        RF-xxx funcionales, RNF-xxx no funcionales
04-historias/         Historias de usuario, agrupadas por épica
05-datos/             Modelo conceptual y diccionario de datos
06-decisiones/        ADR-xxxx: decisiones de producto y arquitectura
07-integraciones/     Sistemas externos (listas restrictivas, fuentes públicas, etc.)
_plantillas/          Plantillas base — copiar, nunca editar en sitio
```

## Convenciones

**Nombres de archivo:** `kebab-case`, sin tildes ni ñ, con el ID por delante cuando el
artefacto lleva ID. Ej: `HU-014-cargar-documentos-de-la-contraparte.md`.

**Identificadores** — únicos, correlativos, nunca se reutilizan ni se renumeran:

| Prefijo | Artefacto | Dónde vive |
|---------|-----------|------------|
| `EP-xxx`  | Épica | `04-historias/EP-xxx-.../EP-xxx-....md` |
| `HU-xxx`  | Historia de usuario | `04-historias/EP-xxx-.../HU-xxx-....md` |
| `RF-xxx`  | Requisito funcional | `03-requisitos/funcionales.md` |
| `RNF-xxx` | Requisito no funcional | `03-requisitos/no-funcionales.md` |
| `FL-xxx`  | Flujo | `02-producto/flujos/` |
| `ADR-xxxx`| Decisión | `06-decisiones/` |
| `PA-xxx`  | Pregunta abierta | `01-descubrimiento/preguntas-abiertas.md` |
| `SUP-xxx` | Supuesto | `01-descubrimiento/supuestos.md` |

**Estados** (en el front-matter de cada artefacto):
`borrador` → `en-revision` → `aprobado` → `implementado` | `descartado`

**Front-matter** obligatorio en historias, épicas, flujos y ADRs:

```yaml
---
id: HU-014
titulo: Cargar documentos de la contraparte
estado: borrador
epica: EP-003
actualizado: 2026-08-21
---
```

## Cómo se escriben las historias

Las historias se generan con la skill **`/user-story-writing`** (plugin
`pm-skills@product-on-purpose`, habilitado en `.claude/settings.json`). No escribir
historias a mano saltándose ese formato: la skill define la estructura, los criterios de
aceptación y el nivel de detalle esperado.

Antes de invocarla, tener a mano:

- la épica o capacidad de la que cuelga (`02-producto/mapa-de-capacidades.md`),
- el actor y su rol (`00-contexto/actores-y-roles.md`),
- los términos de dominio ya definidos (`00-contexto/glosario.md`).

**Definition of Ready** — una historia no pasa a `aprobado` si:

- [ ] no tiene actor, valor y criterios de aceptación verificables,
- [ ] usa términos que no están en el glosario,
- [ ] depende de una `PA-xxx` sin responder (o de un `SUP-xxx` no registrado),
- [ ] no está trazada a una épica,
- [ ] mezcla varios objetivos en una sola historia.

## Reglas al escribir specs

1. **Vacío honesto sobre relleno plausible.** Si no se sabe algo, va `TBD` + una `PA-xxx`.
   Nunca inventar reglas de negocio, umbrales, plazos ni referencias normativas.
2. **La normativa se cita solo si está verificada** contra fuente oficial. Mientras tanto,
   marcar como `(por validar)`.
3. **El glosario manda.** Un mismo concepto se llama igual en todos los documentos; si hace
   falta un término nuevo, primero se agrega al glosario.
4. **Un artefacto, un archivo.** Nada de documentos gigantes con todo dentro.
5. **Datos ficticios** en ejemplos: nunca nombres, cédulas, NIT o casos reales.
6. **Diagramas en Mermaid** dentro del propio Markdown, no imágenes sueltas.
7. Al cerrar una `PA-xxx`, actualizar los documentos que dependían de ella y el `SUP-xxx`
   que la sustituía.

## Flujo de trabajo típico

1. Reunión / input del cliente → nota en `01-descubrimiento/notas/`.
2. Lo que quedó claro → `00-contexto/` y `02-producto/`. Lo que no → `preguntas-abiertas.md`.
3. Capacidad estable → se abre una épica en `04-historias/`.
4. Épica → historias con `/user-story-writing`.
5. Decisión con alternativas y consecuencias → ADR en `06-decisiones/`.

## Git

- Rama `main`; commits en español, imperativo y con el ID por delante:
  `HU-014: agrega criterios de aceptación de carga de documentos`.
- Un commit por artefacto o por conjunto coherente de cambios.
