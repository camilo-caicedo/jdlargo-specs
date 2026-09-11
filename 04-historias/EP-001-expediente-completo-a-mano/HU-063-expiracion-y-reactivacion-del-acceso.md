---
id: HU-063
titulo: Expiración y reactivación del acceso de la contraparte
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-09-11
---

# HU-063 — Expiración y reactivación del acceso de la contraparte

> **Detectada en la auditoría pre-despliegue de `EP-001` (2026-09-11), resuelve `PA-049`.**
> `HU-009` (máquina de estados) y `HU-010` (acceso por enlace) prometían desde el 2026-09-05
> que el expediente pasaba a `Expirado/Pendiente` cuando su enlace vencía, con recordatorios
> automáticos y escalación — y ninguna de las dos lo construyó, porque ningún escenario Gherkin
> propio de esas historias lo exigía. `PA-049` quedó abierta señalando exactamente ese vacío.
> Esta historia lo cierra.

## Historia

**Como** Sistema
**quiero** detectar cuando el enlace de un expediente vence sin que la contraparte haya
terminado de entregar, mover el expediente a un estado visible y avisar a quien debe actuar
**para** que ningún expediente quede varado en silencio esperando un enlace que ya no sirve.

## Contexto

`HU-010` ya resuelve el vencimiento del **enlace**: el token pasa a `expired`, y si alguien
intenta usarlo, el portal lo rechaza con instrucciones para pedir uno nuevo (`HU-010`,
Escenario "Acceso denegado"). Lo que falta es el vencimiento visto desde el **expediente**: hoy
nada distingue, en el listado o el detalle de un expediente, uno cuyo enlace sigue vigente de
uno cuyo enlace lleva semanas vencido sin que nadie lo haya notado — ambos se ven igual, en
`enviada` o `en_diligenciamiento`, hasta que un usuario interno entra a revisar manualmente.

`issueAccessLink` (`HU-010`) ya resuelve la reactivación en sí: emitir un enlace nuevo
reemplaza el activo y no toca ni las afirmaciones declaradas ni los documentos cargados. Lo que
esta historia agrega es la parte que faltaba alrededor de eso: **detectarlo sin que alguien
tenga que acordarse de mirar**, y **avisar** — primero al responsable, después a quien pueda
intervenir si el responsable no reacciona.

## Criterios de aceptación

```gherkin
Escenario: El expediente pasa a Expirado/Pendiente cuando el enlace vence sin avance
  Dado un expediente en "enviada" o "en diligenciamiento" cuyo enlace de acceso activo venció
  Cuando el sistema revisa los expedientes pendientes
  Entonces el expediente transita a "Expirado/Pendiente"
  Y el hecho queda registrado en la bitácora
  Y lo que la contraparte ya había diligenciado y cargado se conserva intacto
```

```gherkin
Escenario: Recordatorio automático al responsable interno
  Dado un expediente en "Expirado/Pendiente"
  Cuando el sistema revisa los expedientes pendientes y el expediente sigue sin un enlace activo
  Entonces se envía un recordatorio por correo al responsable interno del expediente
  Y el envío queda registrado en la bitácora
  Y no se envía un segundo recordatorio el mismo día por el mismo expediente
```

```gherkin
Escenario: Reemitir el enlace conserva el progreso y saca al expediente de Expirado/Pendiente
  Dado un expediente en "Expirado/Pendiente"
  Cuando un usuario con permiso emite un enlace nuevo para ese expediente
  Entonces el expediente transita de vuelta a "en diligenciamiento"
  Y las afirmaciones declaradas y los documentos ya cargados por la contraparte siguen intactos
  Y el enlace anterior queda inválido
```

```gherkin
Escenario: Tras varios recordatorios sin acción, se escala
  Dado un expediente en "Expirado/Pendiente" con 3 recordatorios ya enviados al responsable interno
  Cuando el sistema revisa los expedientes pendientes y el expediente sigue sin un enlace activo
  Entonces se envía un aviso de escalación a los miembros con permiso de decidir de la organización cliente
  Y el aviso queda registrado en la bitácora
  Y no se repite la escalación en cada revisión posterior mientras el expediente siga en ese estado
```

```gherkin
Escenario: Un expediente que ya avanzó no entra en Expirado/Pendiente
  Dado un expediente cuyo enlace original venció, pero que ya transitó a "documentos recibidos" o más adelante
  Cuando el sistema revisa los expedientes pendientes
  Entonces ese expediente no transita a "Expirado/Pendiente"
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la detección de expiración
  Dado expedientes vencidos en "Alfa Ficticia S.A.S." y en "Beta Ficticia S.A.S."
  Cuando el sistema revisa los expedientes pendientes
  Entonces cada expediente transita y notifica dentro de su propia organización cliente
  Y ningún recordatorio ni escalación cruza de una organización a otra
```

## Reglas de negocio

- Solo entran en `Expirado/Pendiente` los expedientes en `enviada` o `en_diligenciamiento` — los
  dos estados donde el avance depende de que la contraparte entre por el enlace. Un expediente
  en cualquier estado posterior ya no depende del enlace original y no se ve afectado, aunque
  ese enlace figure como `expired`.
- La detección y la transición son responsabilidad del **sistema** (`actorType: 'system'`),
  igual que las demás transiciones automáticas de la épica (`HU-010`, `HU-014`) — no depende de
  que un usuario abra la página del expediente, porque nadie tiene por qué volver a abrir un
  expediente que cree resuelto.
- Reemitir el enlace (`HU-010`, ya construido) es la única reactivación: no existe una acción
  separada de "reactivar". Emitir un enlace nuevo sobre un expediente en `Expirado/Pendiente`
  lo regresa a `en_diligenciamiento` en el mismo paso.
- El recordatorio va al responsable interno del expediente (`internal_owner_id`, `HU-008`). La
  escalación, tras un número fijo de recordatorios sin que el expediente cambie de estado, va a
  los miembros de la organización cliente con el permiso de decidir (`dossier:approve`).
- Ni el recordatorio ni la escalación se repiten más de una vez por ciclo de revisión ni se
  duplican el mismo día — evitar el spam es parte de la regla, no un detalle de implementación.
- Esta historia no crea ninguna tabla de estado propia para contar recordatorios: se apoya en la
  bitácora transversal ya existente (`HU-006`) como fuente de verdad de cuántos recordatorios
  lleva un expediente.

## Fuera de alcance

- Configurar por organización cliente cuántos recordatorios se envían antes de escalar, o cada
  cuánto se revisan los expedientes pendientes → Fase 5, junto con el resto de la interfaz de
  administración de la configuración (`HU-007`). Aquí el número de recordatorios y la
  periodicidad son valores fijos del sistema.
- Recordar a la propia contraparte (reenviarle el enlace por su cuenta antes de que un interno
  decida reemitirlo) → no lo pide ningún escenario de esta historia; el recordatorio es siempre
  hacia adentro de la organización cliente.
- Cualquier otro canal de aviso que no sea correo (SMS, notificación en la app) → mismo alcance
  que ya fijó `HU-010` para el enlace mismo.
- El vencimiento y las vigencias de los **documentos** (no del enlace) → Fase 2, ya fuera de
  alcance desde `HU-013`.

## Datos y validaciones

No introduce entidades nuevas. Además del estado y las transiciones del expediente (ver abajo),
consulta lo que ya existe:

| Origen del dato | Historia | Qué aporta |
|---|---|---|
| `dossier_access_tokens.state`/`expires_at` | `HU-010` | Si el enlace activo del expediente venció |
| `dossiers.internal_owner_id` | `HU-008` | A quién recordarle |
| `role_permissions` (permiso `dossier:approve`) | `HU-003` | A quién escalar |
| Bitácora (`audit_log`) | `HU-006` | Cuántos recordatorios lleva un expediente, para decidir si ya toca escalar |

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `dossier_states` (nuevo valor) | Sí | `expirado_pendiente`, no final | No |
| `valid_transitions` (nuevas filas) | Sí | `{enviada, en_diligenciamiento} → expirado_pendiente` (actor sistema, sin permiso de usuario); `expirado_pendiente → en_diligenciamiento` (mismo permiso que ya exige emitir un enlace, `HU-010`) | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 7 (portal de la contraparte, comportamiento esperado ante enlaces
  vencidos), `08-desarrollo/arquitectura-de-aplicacion.md` §"El portal de la contraparte"
- Decisiones: `ADR-0001` (monolito Next.js, sin proceso worker aparte — la revisión periódica se
  implementa como tarea programada dentro del propio despliegue, no como servicio nuevo)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna. `PA-049` **resuelta** por esta misma historia.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-009` (estados y transiciones), `HU-010` (enlace de acceso y su propio
  estado), `HU-008` (responsable interno), `HU-003` (permiso `dossier:approve` para la
  escalación), `HU-006` (bitácora, usada como contador de recordatorios).
- **Habilita a:** nada dentro de `EP-001` — cierra el recorrido de la Fase 1 respecto a enlaces
  vencidos; no bloquea ninguna historia posterior de la épica.
- **Riesgo:** si la revisión periódica no corre (falla el mecanismo que la dispara), ningún
  expediente vencido se detecta y esta historia se vuelve invisible sin que nadie lo note —
  igual que un cron que nunca se configuró. Vale la pena un chequeo operativo aparte del propio
  código (¿corrió hoy?), no solo la lógica de detección en sí.
