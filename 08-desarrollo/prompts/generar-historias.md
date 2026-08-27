---
id: DEV-prompt-historias
estado: vivo
actualizado: 2026-08-25
---

# Prompt para generar historias con Claude Code

Se ejecuta **dentro del repo `jdlargo-specs`**. Reemplazar lo que va entre `<>` para cada épica.

## Prompt completo (primera épica)

```text
Vas a escribir la épica EP-000 (Cimientos) y sus historias de usuario para la
Plataforma JD Largo. Estás en jdlargo-specs, el repositorio de definición.

ANTES DE ESCRIBIR NADA, lee en este orden:
1. CLAUDE.md — convenciones del repo: IDs, estados, front-matter, definition of ready
2. 02-producto/roadmap.md — el faseo. Trabajas SOLO sobre la Fase 0
3. 00-contexto/actores-y-roles.md y 00-contexto/glosario.md — actores y términos permitidos
4. 06-decisiones/ADR-0004-reglas-como-datos.md y ADR-0005-modelo-de-procedencia.md
   — son restricciones duras, no sugerencias
5. 08-desarrollo/arquitectura-de-aplicacion.md — RLS con contexto de usuario, portal externo
6. 05-datos/modelo-conceptual.md — los cuatro planos del modelo
7. 01-descubrimiento/preguntas-abiertas.md y supuestos.md — lo que todavía NO se sabe
8. 04-historias/README.md y las plantillas de _plantillas/
9. El documento fuente del cliente en 01-descubrimiento/entregables-cliente/,
   solo las secciones relevantes a la Fase 0 (§2, §30, §31, §37, §41, §23)

PROCEDIMIENTO — dos pasos, con parada obligatoria entre ellos:

PASO 1 · PROPUESTA. No escribas ningún archivo todavía. Devuélveme en el chat:
- El objetivo de la épica en una frase
- La lista de historias propuestas: ID tentativo, título, actor y una línea de valor
- Qué preguntas abiertas (PA-xxx) afectan a cada historia
- Qué decidiste NO incluir en esta épica y por qué
Después de eso, DETENTE y espera mi aprobación.

PASO 2 · ESCRITURA. Solo cuando yo apruebe explícitamente. Usa la skill
/user-story-writing para cada historia. Crea 04-historias/EP-000-cimientos/ con la
épica y sus historias, y actualiza 04-historias/backlog.md.

REGLAS QUE NO PUEDES ROMPER:
- No inventes requisitos. Si falta información, regístrala como PA nueva en
  01-descubrimiento/preguntas-abiertas.md y deja la historia en estado borrador
  con la dependencia anotada. Un TBD explícito es mejor que un supuesto silencioso.
- Usa solo términos del glosario. Si necesitas uno nuevo, agrégalo al glosario primero.
- Criterios de aceptación verificables, en Gherkin y en español.
- Toda tabla del dominio lleva organization_id y política RLS: la prueba de
  aislamiento entre organizaciones debe aparecer como criterio de aceptación,
  no como un comentario al margen.
- Ninguna historia puede introducir un valor plano donde ADR-0005 exige una
  afirmación con procedencia, ni una regla de cumplimiento escrita en código
  donde ADR-0004 exige configuración versionada.
- Front-matter completo, IDs correlativos, español, kebab-case sin tildes en los
  nombres de archivo.
- Un commit por artefacto, con el ID por delante del mensaje.

SOBRE ESTA ÉPICA EN PARTICULAR: la Fase 0 es de cimientos y no entrega pantallas.
Sus historias son habilitadoras: el actor suele ser el sistema o el propio equipo
de desarrollo. Eso está bien — pero el valor debe expresarse en términos de qué
hace posible o qué riesgo elimina, nunca como "para tener la tabla X".

Al terminar, dime qué preguntas abiertas nuevas registraste y qué quedó en borrador.
```

## Prompt para las épicas siguientes

Una vez que EP-000 esté aprobada, para cada épica posterior basta con:

```text
Escribe la épica <EP-001 · Un expediente completo, a mano> y sus historias,
siguiendo el mismo procedimiento y las mismas reglas que usamos para EP-000
(propuesta primero, esperar aprobación, luego escritura con /user-story-writing).

Alcance: la <Fase 1> del roadmap, nada más.
Lee además del documento del cliente las secciones <§3, §4, Fases 3 a 7, Fase 17,
Fase 18, Fase 19 nivel 1, §38, §44, §46>.
Presta atención especial a <el portal de la contraparte: entra sin cuenta, con
token acotado a un solo expediente, y es superficie pública>.
```

## Mapa de fases, épicas y secciones a leer

| Fase | Épica | Secciones del documento del cliente |
|---|---|---|
| 0 · Cimientos | `EP-000` | §2, §30, §31, §37, §41, Fase 23 |
| 1 · Expediente a mano | `EP-001` | §3, §4, Fases 3-7, 17, 18, 19 (nivel 1), §38, §44, §46 |
| 2 · Extracción | `EP-002` | Fases 8 y 9, §32 |
| 3 · Verificación | `EP-003` | Fases 11, 12 y 13 |
| 4 · Riesgo | `EP-004` | Fases 10, 14, 15 y 16 |
| 5 · Autogestión | `EP-005` | Fases 0, 1 y 2, §41 |
| 6 · Monitoreo | `EP-006` | Fases 20, 21 y 22 |
| C · Comercial | `EP-00C` | `ADR-0002` y §39 |

## Por qué el prompt está construido así

- **La parada entre propuesta y escritura no es opcional.** Sin ella se generan treinta
  historias mediocres de una sentada y revisarlas cuesta más que escribirlas.
- **Se listan los archivos a leer y en qué orden.** Si no, el asistente reconstruye el
  contexto adivinando y gasta tokens redescubriendo lo que ya está decidido.
- **Se nombran `ADR-0004` y `ADR-0005` como restricciones duras.** Son las dos decisiones
  que una historia mal escrita puede violar sin que se note hasta que el código ya existe.
- **Se acota el documento del cliente por secciones.** Son 1.100 líneas: leerlas enteras en
  cada épica es desperdicio.
