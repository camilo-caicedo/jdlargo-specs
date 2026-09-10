---
id: HU-061
titulo: Editar el expediente
estado: implementado
epica: EP-001
prioridad: Should
actualizado: 2026-09-10
---

# HU-061 — Editar el expediente

> **Origen.** Vacío detectado por Camilo el 2026-09-08 auditando la implementación de
> `HU-008`/`HU-010`: existe crear el expediente (`HU-008`) y emitir/reemplazar su enlace de
> acceso (`HU-010`), pero ninguna historia cubre corregir los datos administrativos del
> expediente después de creado — el responsable se equivocó de fecha límite, o el expediente
> quedó asignado a la persona que no era, y hoy no hay forma de arreglarlo salvo tocar la base
> de datos a mano.

## Historia

**Como** Usuario operativo o Analista de Cumplimiento con permiso sobre el expediente
**quiero** corregir el responsable interno y la fecha límite de un expediente ya creado
**para** no quedar atado a un error de captura sin tener que cancelar y recrear el expediente.

## Contexto

`HU-008` congela, al crear el expediente, la versión de configuración y con ella la matriz de
requisitos (`ADR-0004`) — eso es intencional y no se reabre aquí. Lo que esta historia cubre es
solo el **dato administrativo**, no normativo: quién es el responsable interno
(`dossiers.internal_owner_id`) y cuándo vence (`dossiers.deadline`). Tocar el tipo de
contraparte, el estándar o la versión de configuración citada equivale a abrir un expediente
distinto — por eso queda fuera (ver Fuera de alcance).

## Criterios de aceptación

```gherkin
Escenario: Corregir el responsable interno de un expediente abierto
  Dado un expediente que no está en un estado final ("cerrada")
  Cuando un usuario con permiso "dossier:edit" cambia el responsable interno
  Entonces el expediente queda asignado al nuevo responsable
  Y el cambio queda registrado en la bitácora con quién lo hizo y cuándo

Escenario: Corregir la fecha límite de un expediente abierto
  Dado un expediente que no está en un estado final ("cerrada")
  Cuando un usuario con permiso "dossier:edit" cambia la fecha límite
  Entonces el expediente refleja la nueva fecha
  Y el cambio queda registrado en la bitácora

Escenario: Un expediente cerrado no se puede editar
  Dado un expediente en estado "cerrada"
  Cuando alguien intenta cambiar su responsable interno o su fecha límite
  Entonces el sistema rechaza el cambio como error de dominio
  Y no se modifica ningún dato

Escenario: Sin el permiso no hay edición
  Dado un usuario sin el permiso "dossier:edit" sobre la organización
  Cuando abre el detalle de un expediente
  Entonces no ve ningún control para editar el responsable interno ni la fecha límite
```

## Reglas de negocio

- Editable mientras el expediente **no** esté en el estado final `cerrada`. No se introduce
  aquí ninguna restricción adicional por estado intermedio — igual de editable en `borrador`,
  `enviada`, `en_diligenciamiento`, `en_revision`, etc.
- El permiso es el mismo `dossier:edit` que ya gatea emitir/reemplazar el enlace de acceso
  (`HU-010`) — no se crea un permiso nuevo en el catálogo cerrado de `HU-003`.
- Todo cambio queda en la bitácora (`ADR-0007`/`HU-006`): quién, qué campo, valor anterior,
  valor nuevo, cuándo.
- El nuevo responsable interno debe ser una membresía activa de la misma organización cliente
  — mismo criterio que usa hoy `HU-008` al asignar el responsable en la creación.

## Fuera de alcance

- Cambiar el tipo de contraparte, el estándar o la versión de configuración del expediente —
  eso invalida la matriz de requisitos ya congelada; equivale a abrir un expediente nuevo.
- Editar los datos declarados por la contraparte (nombre, identificación) — eso es afirmación
  con procedencia `declarado` (`ADR-0005`), se corrige por el flujo de `HU-014` (solicitud de
  correcciones), no reescribiéndola directo.
- Reenviar o regenerar el enlace de acceso — ya cubierto por `HU-010`.
- Un historial de auditoría visible en pantalla para estos cambios puntuales — la bitácora ya
  los registra (`HU-006`); mostrarlos en la UI del expediente es extensión de `HU-016`, no de
  esta historia.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `dossier.internal_owner_id` | No (puede quedar sin asignar) | Si se envía, debe ser una membresía activa de la organización | No |
| `dossier.deadline` | No | Fecha válida; sin regla de negocio propia sobre mínimos/máximos | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Decisiones: `ADR-0004` (por qué la matriz no se reabre), `ADR-0007` (bitácora)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-008` (existe el expediente y el campo `internal_owner_id`/`deadline`),
  `HU-009` (el estado `cerrada` como límite de edición), `HU-003` (permiso `dossier:edit` ya
  existente en el catálogo).
- **Habilita a:** nada aguas abajo directamente — cierra un hueco operativo detectado tarde.
- **Riesgo:** ninguno de negocio. El único riesgo es de alcance: no dejar que esta historia
  crezca hacia editar datos declarados o la matriz — ver Fuera de alcance.
