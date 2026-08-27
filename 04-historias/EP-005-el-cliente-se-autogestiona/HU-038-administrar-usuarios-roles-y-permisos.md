---
id: HU-038
titulo: Administrar usuarios, roles y permisos
estado: borrador
epica: EP-005
prioridad: Must
actualizado: 2026-08-27
---

# HU-038 — Administrar usuarios, roles y permisos

## Historia

**Como** Administrador de una organización cliente
**quiero** invitar a mi equipo, asignarle roles y ajustar qué puede hacer cada uno
**para** que la segregación de funciones que exige mi política se refleje en la plataforma sin
que nadie externo tenga que tocarla.

## Contexto

`HU-001` construyó organizaciones, cuentas y membresías; `HU-003` convirtió la matriz de permisos
en configuración versionada. Faltaba lo que la Fase 0 no entregaba por decisión: **las pantallas**
—invitar, aceptar, asignar, revocar.

La §30 es la referencia: la matriz es un ejemplo base y cada cliente ajusta según su propia
segregación de funciones. Y hay una tensión que conviene resolver explícitamente en el diseño: el
Administrador administra usuarios, pero **quién puede aprobar, decidir o modificar la metodología
es una decisión de cumplimiento**, no de administración de sistemas.

## Criterios de aceptación

```gherkin
Escenario: Invitar a una persona a la organización cliente
  Dado un Administrador con permiso de administrar usuarios
  Cuando invita a una persona indicando su correo y su rol
  Entonces se le envía una invitación con vencimiento
  Y la persona queda como miembro solo cuando la acepta
  Y la invitación y su aceptación quedan registradas en la bitácora
```

```gherkin
Escenario: Cambiar el rol de un miembro
  Dado un miembro con un rol asignado
  Cuando el Administrador le asigna otro rol
  Entonces sus permisos cambian desde ese momento
  Y queda registrado quién hizo el cambio, cuándo y qué rol tenía antes
```

```gherkin
Escenario: Revocar una membresía
  Dado un miembro activo
  Cuando el Administrador revoca su membresía
  Entonces pierde el acceso de inmediato
  Y sus acciones anteriores siguen registradas en la bitácora sin alteración
  Y la revocación queda registrada
```

```gherkin
Escenario: Ajustar la matriz de permisos
  Dado un borrador de configuración
  Cuando el Administrador cambia qué permisos tiene un rol
  Entonces el cambio solo rige tras publicar la versión
  Y las acciones anteriores conservan la versión de permisos con la que se evaluaron
```

```gherkin
Escenario: No se puede dejar la organización sin administrador
  Dado una organización cliente con un solo miembro con rol de Administrador
  Cuando se intenta revocarle la membresía o quitarle ese rol
  Entonces la operación es rechazada
  Y se indica que debe existir al menos un Administrador
```

```gherkin
Escenario: No se puede otorgar un permiso que quien lo otorga no tiene
  Dado un Administrador cuyo rol no incluye el permiso de decidir sobre vinculaciones
  Cuando intenta otorgarse a sí mismo ese permiso
  Entonces la operación es rechazada
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Un usuario en varias organizaciones clientes
  Dado una persona miembro de dos organizaciones clientes con roles distintos
  Cuando actúa en cada una
  Entonces sus permisos son los de esa organización cliente
  Y ninguna administración de una afecta a su membresía en la otra
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la administración de usuarios
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta o modifica miembros con su contexto de usuario propagado
  Entonces solo ve y modifica los de "Alfa Ficticia S.A.S."
  Y no puede ver los miembros de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Invitar no otorga acceso: la persona es miembro cuando **acepta**. La invitación vence.
- Cambios de rol y revocaciones tienen efecto inmediato y quedan registrados con el valor anterior
  y el nuevo (§23).
- **La membresía se revoca, no se borra** (`HU-001`): la bitácora tiene que seguir teniendo a
  quién atribuir lo pasado.
- Los cambios en la **matriz de permisos** son configuración versionada y rigen al publicar
  (`HU-003`). Los cambios de **rol de una persona** son operativos y rigen de inmediato.
- Una organización cliente no puede quedarse sin ningún miembro con rol de Administrador.
- **Nadie puede otorgar un permiso que no tiene.** Es la barrera mínima contra la escalada de
  privilegios dentro de una organización cliente.
- Un usuario puede pertenecer a varias organizaciones clientes con roles distintos (`PA-013`,
  resuelta), y cada una administra la suya.

## Fuera de alcance

- El inicio de sesión, la recuperación de contraseña y el segundo factor del usuario interno.
- El inicio de sesión único empresarial, descartado en `ADR-0001` mientras nadie lo exija.
- La creación de roles nuevos, mientras `PA-024` no defina si el cliente puede crearlos o solo
  ajustar los seis definidos.
- La administración de la contraparte como usuario: no lo es, entra por enlace (`HU-010`).
- Grupos, equipos y jerarquías de aprobación.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `invitacion.correo` | Sí | Correo válido | Sí (dato personal) |
| `invitacion.rol_id` | Sí | Rol vigente de esa organización cliente | No |
| `invitacion.expira_en` | Sí | Momento futuro | No |
| `invitacion.estado` | Sí | `pendiente` \| `aceptada` \| `vencida` \| `revocada` | No |
| `membresia.rol_id` | Sí | Rol vigente de la organización cliente | No |
| `membresia.estado` | Sí | `activa` \| `revocada` | No |
| Administradores mínimos | Sí | Al menos uno activo por organización cliente | No |
| Otorgamiento de permisos | Sí | Nadie otorga lo que no tiene | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: §30, §23
- Decisiones: `ADR-0004` (el control de acceso es configuración), `ADR-0001` (sin inicio de sesión
  único empresarial)
- Historias: pone interfaz a `HU-001` y `HU-003`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-024` — bloqueante: si el cliente puede crear roles propios o solo
  ajustar los definidos, y quién asigna roles. `PA-031` — el envío de invitaciones por correo.
  **Queda en `borrador`.**
- **Supuestos:** `SUP-005`.
- **Depende de:** `HU-001`, `HU-003`, `HU-034`.
- **Habilita a:** que un cliente nuevo se ponga en marcha sin intervención nuestra.
- **Riesgo:** el Administrador es un rol técnico y los permisos que administra son de
  cumplimiento. Si nada lo limita, el rol que gestiona usuarios acaba pudiendo otorgarse a sí
  mismo la facultad de aprobar vinculaciones, y la segregación de funciones de la §30 se vuelve
  decorativa.
