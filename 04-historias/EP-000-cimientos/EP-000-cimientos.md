---
id: EP-000
titulo: Cimientos
estado: borrador
capacidad: CAP-00
actualizado: 2026-08-27
---

# EP-000 — Cimientos

> **Esta épica no entrega pantallas y no se puede demostrar.** Es la Fase 0 del
> `02-producto/roadmap.md`: las cuatro cosas que, añadidas después, no serían una ampliación
> sino una reescritura. Conviene decirlo así de claro cuando se presente el plan.

## Objetivo

Dejar montado el sustrato del que depende todo lo demás —**aislamiento entre organizaciones,
procedencia del dato, configuración versionada y bitácora**— de modo que el primer
expediente que exista nazca ya auditable, sin necesidad de corregirlo hacia atrás.

## Por qué existe

Tres restricciones del proyecto no admiten ser diferidas:

| Restricción | Origen | Qué pasa si se aplaza |
|---|---|---|
| Ningún dato de una organización cliente es visible desde otra | §31, `ADR-0001` | Una fuga entre clientes del SaaS. No es un error visible: es un cliente viendo lo ajeno |
| Todo dato del expediente lleva su procedencia | §2, `ADR-0005` | Si el primer expediente guarda valores planos, la conciliación y la auditoría no se pueden montar encima |
| La regla de cumplimiento es configuración versionada, no código | §41, `ADR-0004` | Un expediente sin la versión con la que fue evaluado es inauditable, y eso no se arregla hacia atrás |

La bitácora (§23) es la cuarta: **desde la primera tabla**, porque una bitácora que empieza
tarde deja un tramo del historial sin explicación.

## Alcance

**Incluye:**

- Organizaciones clientes, cuentas de usuario y pertenencia con rol (`HU-001`).
- Aislamiento entre organizaciones por política de base de datos, con el contexto de usuario
  propagado en cada transacción, y su prueba (`HU-002`).
- Permisos por rol como configuración de la organización, ajustable sin desplegar código
  (`HU-003`).
- Publicación de versiones de configuración inmutables, con puntero de configuración vigente
  (`HU-004`).
- Registro de afirmaciones con origen, sin sobrescritura (`HU-005`).
- Bitácora de solo inserción, transversal (`HU-006`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Sujetos: Persona, Organización, Relación | Fase 1 | Sin expediente no hay a quién describir |
| Solicitud, expediente y su máquina de estados (§38) | Fase 1 | El expediente se estrena en la Fase 1; modelarlo antes es especular |
| Conciliación y cálculo del valor vigente entre afirmaciones | Fase 2 | En Fase 0 solo existe el origen `declarado`: no hay contradicción posible todavía. `HU-005` deja la estructura, la precedencia queda en `PA-027` |
| Portal de la contraparte y su acceso por enlace | Fase 1 | Segunda superficie, con reglas propias. Se estrena con el expediente que la acota |
| Interfaz de administración de la configuración | Fase 5 | Decisión explícita del roadmap: hasta ahí la configuración se carga a mano |
| Registro de ejecución de IA (§32) | Fase 2 | `HU-006` deja previstos los campos en la bitácora; el módulo no se construye |
| Cuotas, medición de consumo y facturación | Fase C | `ADR-0002` |
| Invitaciones, pantallas de usuarios, recuperación de contraseña | Fase 1 o posterior | La Fase 0 no entrega pantallas |
| Migraciones, integración continua, entorno | — | Plomería de implementación, no definición de producto |

## Actores involucrados

- **Sistema** — ejecuta el aislamiento, escribe la bitácora y conserva la procedencia. Nunca decide.
- **Administrador** — configura los permisos de su organización.
- **Oficial de Cumplimiento** — publica versiones de configuración.
- **Auditor / Consulta** — es quien consume la bitácora, aunque su pantalla llegue después.
- **Equipo de desarrollo** — destinatario real de las historias habilitadoras.

## Criterios de éxito

Medibles, y todos verificables sin interfaz:

1. **Cero lecturas cruzadas.** Existe una prueba automatizada por cada tabla del dominio que
   demuestra que un usuario de la organización A no lee ni escribe filas de la B. Una tabla
   sin esa prueba no se considera terminada.
2. **Cero valores planos en el expediente.** No existe ninguna operación que escriba un dato
   del expediente sin origen asociado.
3. **Cero `UPDATE` sobre configuración publicada.** Intentar modificar una versión publicada
   falla; la única vía es publicar una versión nueva.
4. **Cero acciones sin rastro.** Toda escritura sobre una tabla del dominio deja una entrada
   en la bitácora con actor, momento, valor anterior y nuevo.
5. La bitácora responde, para cualquier fila, **quién la creó o cambió, cuándo, desde dónde y
   bajo qué versión de configuración**, sin construir un reporte aparte.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-001` | Organizaciones, cuentas de usuario y pertenencia | Must | borrador |
| `HU-002` | Aislamiento entre organizaciones con contexto de usuario | Must | borrador |
| `HU-003` | Permisos por rol como configuración de la organización | Must | borrador |
| `HU-004` | Publicación de versiones de configuración inmutables | Must | borrador |
| `HU-005` | Registro de afirmaciones con procedencia | Must | borrador |
| `HU-006` | Bitácora inmutable transversal | Must | borrador |

Orden sugerido: `HU-001` → `HU-002` → `HU-006` → `HU-003` → `HU-004` → `HU-005`. La bitácora
se adelanta a propósito: cada historia posterior la usa, y una bitácora retroajustada deja un
tramo ciego.

## Dependencias

- **Preguntas abiertas:** `PA-009` (retención y trazabilidad), `PA-017` y `PA-018` (quién
  configura y cuánto), `PA-024` (alcance de la configuración de roles), `PA-025` (efecto de
  publicar una versión sobre lo que está en curso), `PA-026` (nivel de inmutabilidad exigido a
  la bitácora), `PA-027` (precedencia entre afirmaciones).
- **Supuestos:** `SUP-002` (multiempresa, confirmado §31), `SUP-005` (todo es una organización).
- **Decisiones:** `ADR-0001` (RLS, Postgres, Drizzle), `ADR-0004` (configuración versionada),
  `ADR-0005` (procedencia).
- **Otras épicas:** ninguna. `EP-000` no depende de nadie; todas las demás dependen de ella.

## Riesgo abierto

`PA-025` y `PA-026` son de alto impacto y afectan al modelo de datos, no a la interfaz. Se
pueden empezar `HU-001`, `HU-002`, `HU-003` y `HU-005` sin ellas, pero **`HU-004` y `HU-006`
no deberían pasar de `borrador`** hasta tener respuesta: elegir mal ahí se paga rehaciendo la
tabla y todo lo que ya escribió en ella.
