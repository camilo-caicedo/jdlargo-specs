---
id: HU-056
titulo: Invitar miembros a la organización
estado: en-revision
epica: EP-001
prioridad: Must
actualizado: 2026-09-07
---

# HU-056 — Invitar miembros a la organización

> **Origen.** Igual que `HU-055`: vacío detectado al auditar la Fase 1, no historia del
> descubrimiento original. `HU-001` excluyó expresamente "flujo de invitación por correo,
> aceptación y recuperación de contraseña" de su alcance, dejando dicho que las membresías
> nacían "por migración o por operación de sistema" en la Fase 0. Sin esta historia, cada
> persona nueva de una organización cliente necesitaría una operación manual de sistema para
> siempre — insostenible más allá del primer cliente ancla.

## Historia

**Como** Administrador de una organización cliente
**quiero** invitar a alguien por correo a mi organización con un rol determinado
**para** que se una a la plataforma sin que yo tenga que crearle la cuenta a mano ni pedirle
credenciales por fuera del sistema.

## Contexto

`HU-001` ya modela la membresía (`Usuario › Membresía › Organización › Roles`) y `HU-003` ya
modela los roles y permisos como configuración de la organización. Falta el puente entre las
dos: cómo llega una persona nueva a tener una membresía sin que un desarrollador la inserte a
mano en la base de datos.

El patrón de enlace de un solo uso, con expiración y correo transaccional, **ya existe en el
repo** desde `HU-010` (acceso de la contraparte): token con huella almacenada, no el valor;
expiración configurable; un enlace vigente a la vez por invitación pendiente. Esta historia
reutiliza ese patrón para un caso distinto — invitar a un colega, no dar acceso a una
contraparte — pero es el mismo mecanismo, no uno nuevo.

## Criterios de aceptación

```gherkin
Escenario: Invitar a una persona nueva en la plataforma
  Dado un Administrador de "Alfa Ficticia S.A.S." con permiso de gestionar membresías
  Cuando invita al correo "nueva@ejemplo.com" con el rol de Analista de Cumplimiento
  Entonces queda registrada una invitación pendiente para ese correo, esa organización cliente y ese rol
  Y se envía un correo a "nueva@ejemplo.com" con el enlace de aceptación
  Y la invitación queda registrada en la bitácora
```

```gherkin
Escenario: Aceptar una invitación siendo persona nueva en la plataforma
  Dado una invitación pendiente y vigente para "nueva@ejemplo.com" en "Alfa Ficticia S.A.S."
  Y que "nueva@ejemplo.com" no tiene ninguna cuenta todavía
  Cuando esa persona abre el enlace y establece su contraseña
  Entonces queda creada su cuenta
  Y queda con membresía activa en "Alfa Ficticia S.A.S." con el rol que se le asignó
  Y la invitación pasa a estado aceptada
```

```gherkin
Escenario: Aceptar una invitación teniendo ya una cuenta
  Dado una invitación pendiente y vigente para "existente@ejemplo.com" en "Beta Ficticia S.A.S."
  Y que "existente@ejemplo.com" ya tiene cuenta y una membresía activa en "Alfa Ficticia S.A.S."
  Cuando esa persona abre el enlace e inicia sesión con su contraseña ya existente
  Entonces su cuenta gana una membresía activa nueva en "Beta Ficticia S.A.S."
  Y su membresía existente en "Alfa Ficticia S.A.S." no se altera
  Y no se crea una segunda cuenta ni una segunda identidad para esa persona
```

```gherkin
Escenario: Una invitación expirada no se puede aceptar
  Dado una invitación cuya fecha de expiración ya pasó
  Cuando la persona invitada abre el enlace
  Entonces no se le crea ni se le activa ninguna membresía
  Y ve un mensaje indicando que la invitación expiró y a quién pedir una nueva
```

```gherkin
Escenario: Revocar una invitación pendiente
  Dado una invitación pendiente para "nueva@ejemplo.com" en "Alfa Ficticia S.A.S."
  Cuando el Administrador la revoca antes de que se acepte
  Entonces el enlace de esa invitación deja de dar acceso
  Y la revocación queda registrada en la bitácora con quién la ejecutó y cuándo
```

```gherkin
Escenario: Reenviar una invitación invalida la anterior
  Dado una invitación pendiente para "nueva@ejemplo.com" en "Alfa Ficticia S.A.S."
  Cuando el Administrador la reenvía
  Entonces se emite una invitación nueva con su propio enlace
  Y el enlace de la invitación anterior deja de dar acceso
  Y solo hay una invitación pendiente vigente por correo y organización cliente a la vez
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las invitaciones
  Dado un Administrador miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta las invitaciones pendientes con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
  Y un intento de crear o revocar una invitación de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

```gherkin
Escenario: Invitar exige el permiso correspondiente
  Dado un usuario cuyo rol vigente no incluye el permiso de gestionar membresías
  Cuando intenta invitar a alguien
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

## Reglas de negocio

- **Nunca se crea ni se activa una membresía sin que la persona invitada la haya aceptado
  explícitamente.** Una invitación pendiente no es una membresía.
- El rol se asigna al momento de invitar, y tiene que existir en el catálogo de roles vigente
  de esa organización cliente (`HU-003`). No se inventa un rol en el momento de aceptar.
- Una invitación vale para **un correo y una organización cliente**. La misma persona puede
  tener invitaciones pendientes distintas en organizaciones clientes distintas al mismo tiempo.
- Solo hay una invitación pendiente vigente por combinación correo–organización cliente.
  Reenviar invalida la anterior, mismo principio que "un enlace vigente a la vez" de `HU-010`.
- Si la persona invitada ya tiene cuenta en la plataforma (con o sin otras membresías), aceptar
  **agrega** una membresía a su identidad existente — nunca crea una segunda cuenta para el
  mismo correo (`user.email` ya es único, `HU-001`).
- Invitar requiere permiso explícito — `memberships:manage` del catálogo cerrado (`HU-003`).
  Según la matriz base lo tiene el Administrador.
- La expiración de la invitación es configurable, con **7 días** como valor por defecto —
  tiempo suficiente para que la persona vea el correo y actúe, sin dejar el enlace abierto
  indefinidamente. Es un valor de arranque del arquitecto, no una cifra normativa: se corrige
  cambiando la configuración, nunca el esquema ni el código, si Juan David pide otra.

## Fuera de alcance

- Registro público sin invitación — sigue sin existir en el MVP (`PA-038`).
- Editar o quitar el rol de un miembro ya activo, o revocar su membresía → ya cubierto por el
  modelo de `HU-001`/`HU-003`; la pantalla para hacerlo con interfaz es `HU-038`, Fase 5.
- Recuperar contraseña → `HU-055`/`HU-057` (la persona invitada que ya tenía cuenta usa su
  contraseña existente; si la olvidó, sigue el flujo de `HU-057`, no uno propio de invitación).
- Invitar a la contraparte — no aplica; la contraparte entra por `HU-010`, nunca tiene cuenta.
- Recordatorio automático de invitación pendiente que va a expirar → mismo patrón que los
  recordatorios de `HU-045`, Fase 6, no aquí.
- Límite de cuántas invitaciones puede enviar una organización cliente (relacionado con cupos)
  → Fase C, `ADR-0002`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `invitation.organization_id` | Sí | Organización cliente existente | No |
| `invitation.email` | Sí | Correo válido | Sí (dato personal) |
| `invitation.role_id` | Sí | Rol vigente de esa organización cliente (`HU-003`) | No |
| `invitation.token` | Sí | Aleatorio, de longitud suficiente; se almacena su huella, no el valor | Sí |
| `invitation.expires_at` | Sí | Momento futuro; duración configurable, 7 días por defecto | No |
| `invitation.state` | Sí | `pending` \| `accepted` \| `expired` \| `revoked` \| `replaced` | No |
| `invitation.invited_by` | Sí | Usuario con permiso `memberships:manage` en esa organización cliente | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: §30 (roles), §31 (multiempresa)
- Decisiones: `ADR-0001` §7 (identidad global y membresía)
- Requisito funcional: `RF-056`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-001` (usuarios y membresías), `HU-003` (catálogo de roles a asignar),
  `HU-055` (la persona invitada necesita poder iniciar sesión después de aceptar).
- **Habilita a:** que una organización cliente crezca más allá de su primer Administrador sin
  intervención manual de sistema.
- **Riesgo:** si aceptar una invitación llegara a crear una cuenta nueva para un correo que ya
  existe, se duplicaría la identidad de una persona y se rompería el modelo de `HU-001`
  ("identidad global, no una por organización cliente"). El escenario de "aceptar teniendo ya
  cuenta" existe justamente para dejar esto probado, no solo enunciado.
