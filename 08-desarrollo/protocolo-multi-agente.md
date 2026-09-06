---
id: DEV-protocolo-multiagente
estado: propuesto
actualizado: 2026-09-06
---

# Protocolo de orquestación multi-agente: Claude Code y Antigravity

Marco operativo para la colaboración entre **Claude Code** y **Antigravity** al construir
`jdlargo-api`/`jdlargo-web`. Es el mismo patrón que ya usa el equipo en `Coffea/` (ver
`Coffea/coffea-specs/docs/specs/protocolo-multi-agente.md`), adaptado a este proyecto: un
solo desarrollador (Camilo) apoyado en dos agentes con roles fijos, en vez de un backend y
un frontend separados con contrato OpenAPI entre ellos.

> **Complemento de** `08-desarrollo/arquitectura-de-aplicacion.md` §"Metodología de
> construcción: Superpowers" y de `AGENTS.md` (raíz del espacio de trabajo) §8. Ese documento
> ya decidió Superpowers como metodología; este documento decide **cómo se reparte entre dos
> agentes**, no si se usa.

## 1. Por qué dos agentes y no uno

Explota la asimetría real entre las dos herramientas:

| Dimensión | Claude Code | Antigravity |
|---|---|---|
| **Ventana de contexto** | Más chica, sensible a saturarse | Sustancialmente mayor: puede absorber exploración de código y bucles de build/test sin agotarse |
| **Fortaleza cognitiva** | Razonamiento arquitectónico, contratos limpios, detectar sutilezas de las reglas de negocio (`ADR-0004`, `ADR-0005`) | Búsqueda exhaustiva en código, bucles de compilación y prueba, corrección iterativa |
| **Rol asignado** | **Arquitecto / Tech Lead** | **Constructor / QA** |

## 2. Matriz de responsabilidades

```mermaid
sequenceDiagram
    autonumber
    actor Dev as Camilo
    participant Claude as Claude Code (Arquitecto)
    participant Plan as Plataforma-jdlargo/Planes/&lt;tarea&gt;.md
    participant AGY as Antigravity (Constructor)
    participant Git as jdlargo-api / jdlargo-web

    Dev->>Claude: Pide construir HU-xxx
    Note over Claude: Lee jdlargo-specs/ (HU, RF, ADR)<br/>diseña el contrato, no implementa
    Claude->>Plan: Crea archivo de plan único con el blueprint
    Dev->>AGY: Ordena ejecutar el plan
    Note over AGY: Explora el repo (ventana grande)<br/>implementa con TDD<br/>corre build y tests en bucle
    AGY->>Plan: Marca los pasos completados [x]
    AGY-->>Dev: Notifica suite 100% verde
    Dev->>Claude: Pide auditar el git diff
    Claude->>Git: Audita el diff contra el contrato y los criterios Gherkin de la HU
    Claude->>Plan: Mueve el archivo a Planes/Terminados/
    Claude-->>Dev: Aprobación para push
```

### Rol 1 — Claude Code (el Arquitecto)

**Qué hace:**
1. Analiza la `HU-xxx`/`RF-xxx` contra `jdlargo-specs/` — glosario, ADR aplicables, criterios
   de aceptación en Gherkin que la historia ya trae.
2. Define contratos: tipos, forma de la Server Action/Route Handler, esquema Zod, alcance de
   RLS.
3. Redacta el plan de ejecución en un archivo nuevo y único bajo
   `Plataforma-jdlargo/Planes/` (nunca dentro de un repo), con tareas atómicas.
4. Audita el `git diff` final antes de cada merge, contra el contrato y contra los
   escenarios Gherkin de la historia.
5. Tras aprobar, mueve el archivo a `Planes/Terminados/` — esa es la constancia de auditoría.

**Qué NO hace:**
- No corre `npm run build`, la suite completa de pruebas, ni instalaciones grandes de
  paquetes.
- No escribe la implementación completa salvo que se pida explícitamente.
- No escribe el plan dentro de `jdlargo-specs/`, `jdlargo-api/` ni `jdlargo-web/`.

### Rol 2 — Antigravity (el Constructor / Tester)

**Qué hace:**
1. Lee el plan desde su archivo en `Planes/`.
2. Usa su ventana grande para explorar dependencias en `jdlargo-api`/`jdlargo-web`.
3. Implementa con TDD (ciclo rojo-verde-refactor): la prueba sale de los escenarios Gherkin
   de la `HU-xxx` antes que el código.
4. Corre pruebas y build en bucle hasta el 100 % en verde, autocorrigiéndose ante fallos sin
   detenerse a preguntar.
5. Actualiza los checkboxes en el mismo archivo de plan.

**Qué NO hace:**
- No altera decisiones arquitectónicas ni contratos sin dejarlo escrito en "Notas y
  bloqueos" del plan.
- No mueve el archivo a `Terminados/` — eso es exclusivo de Claude Code, después de auditar.
- No comitea código roto o sin probar.

## 3. Los dos agentes corren Superpowers

No es "Claude con Superpowers, Antigravity con otra metodología": es la misma disciplina en
los dos lados, cada uno con el subconjunto de skills que le toca por rol.

| Agente | Skills que usa | Para qué |
|---|---|---|
| Claude Code | `brainstorming`, `writing-plans`, `requesting-code-review` | Diseñar el blueprint cuando el requisito es ambiguo, estructurar el plan, auditar el diff de Antigravity |
| Antigravity | `test-driven-development`, `executing-plans`, `systematic-debugging`, `using-git-worktrees` | Implementar con TDD, ejecutar el plan paso a paso, diagnosticar fallos de build/test, aislar trabajo en curso |

**Instalación:**
- Claude Code ya lo trae como plugin marketplace (`/plugin install
  superpowers@claude-plugins-official`), decidido en `arquitectura-de-aplicacion.md`.
- Antigravity lo carga como extensión de su propio harness — el plugin ya incluye un mapeo
  de herramientas específico para Antigravity CLI (`skills/using-superpowers/references/
  antigravity-tools.md` dentro del propio Superpowers), así que no hace falta adaptarlo a
  mano ni escribir skills paralelas.

## 4. Protocolo de memoria compartida (`Plataforma-jdlargo/Planes/`)

Un archivo de plan por tarea, nombrado según la historia — p. ej.
`HU-023-catalogo-de-fuentes-externas.md` — nunca el genérico `TASK_PLAN.md`. Plantilla en
`Plataforma-jdlargo/Planes/TASK_PLAN_BASE.md`, con una sección de trazabilidad a `HU-xxx`/
`RF-xxx` que no existe en la versión de Coffea (aquí cada historia ya trae sus criterios de
aceptación en Gherkin y su estado de *Definition of Ready* — el plan los cita, no los
reinventa). Al terminar y ser auditado, el archivo se mueve a `Planes/Terminados/`.

## 5. Convención de commits

A diferencia de Coffea (que usa un prefijo `[Claude Code] -` / `[Antigravity] -` porque su
convención de commits no traía ya un ID por delante), aquí se mantiene la convención que
`jdlargo-specs/CLAUDE.md` ya define — `ID: descripción imperativa`, en español, un commit
por artefacto o cambio coherente — y se añade solo un trailer que identifica al agente
autor, igual que ya hacen las sesiones de Claude Code en este espacio de trabajo. No se
bifurca en dos formatos de mensaje: el ID (`HU-xxx`, `RF-xxx`) ya identifica de qué trata el
commit sin necesidad de un prefijo de herramienta.

## 6. Prompts base de cada sesión

- **Claude Code (Arquitecto):** `08-desarrollo/prompts/claude-architect.prompt.md`
- **Antigravity (Constructor):** `08-desarrollo/prompts/antigravity-builder.prompt.md`

## 7. Lo que este documento no repite

La instalación de herramientas CLI, alias de terminal y multiplexación con `tmux` para
correr ambos agentes en paralelo ya está documentada, paso a paso, en
`Coffea/coffea-specs/docs/specs/protocolo-multi-agente.md` §4 y §6, en la misma máquina de
este equipo. No se duplica aquí: solo cambia qué carpeta se clona y qué plan se lee, que es
lo que cubren las secciones de arriba.
