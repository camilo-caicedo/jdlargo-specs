---
id: HU-001
titulo: Organizaciones, cuentas de usuario y pertenencia
estado: borrador
epica: EP-000
prioridad: Must
actualizado: 2026-08-27
---

# HU-001 — Organizaciones, cuentas de usuario y pertenencia

## Historia

**Como** equipo de desarrollo
**quiero** que toda persona que actúe sobre la plataforma esté identificada y adscrita a una
organización cliente con un rol
**para que** cada acción tenga a quién atribuirse y exista el sujeto sobre el que se apoyan el
aislamiento entre organizaciones y la bitácora.

## Contexto

Es la historia que hace posibles a las otras cinco. Sin una identidad verificable no hay
"quién" que registrar en la bitácora (§23) ni contexto que propagar para el aislamiento
entre organizaciones (§31).

Dos decisiones ya tomadas la delimitan:

- **Todo es una organización desde el día uno** (`ADR-0001`, `SUP-005`). El usuario individual
  no es un caso especial: es una organización cliente de un miembro. No hay migración de
  persona a empresa, hay una invitación.
- **Un usuario puede pertenecer a varias organizaciones clientes** (`PA-013`, resuelta), con un
  rol distinto en cada una. El caso real es el consultor que atiende a varios clientes.

La contraparte **no aparece aquí**: no tiene cuenta ni contraseña, entra por enlace acotado a
un expediente (`00-contexto/actores-y-roles.md`, nota de diseño). Es una superficie distinta y
llega con la Fase 1.

## Criterios de aceptación

```gherkin
Escenario: Crear una organización cliente con su primer miembro
  Dado que no existe ninguna organización cliente llamada "Transportes Ficticios S.A.S."
  Cuando se crea esa organización cliente junto con la cuenta de su primer usuario
  Entonces la organización cliente queda registrada con identificador propio
  Y ese usuario queda como miembro de ella con el rol de Administrador
  Y la creación queda registrada en la bitácora con el actor, el momento y el origen de la petición
```

```gherkin
Escenario: Un usuario pertenece a dos organizaciones clientes con roles distintos
  Dado un usuario con cuenta activa
  Y dos organizaciones clientes distintas, "Alfa Ficticia S.A.S." y "Beta Ficticia S.A.S."
  Cuando se le otorga membresía en "Alfa Ficticia S.A.S." con el rol de Analista de Cumplimiento
  Y membresía en "Beta Ficticia S.A.S." con el rol de Auditor
  Entonces el usuario tiene dos membresías independientes
  Y el rol que aplica en cada momento depende de la organización cliente en la que esté actuando
  Y ningún permiso de una organización cliente surte efecto en la otra
```

```gherkin
Escenario: Autenticarse no otorga por sí solo acceso a ninguna organización cliente
  Dado un usuario con cuenta activa y sin ninguna membresía
  Cuando se autentica correctamente
  Entonces obtiene una sesión válida
  Y no puede leer ni escribir dato alguno de ninguna organización cliente
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las tablas de esta historia
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Y una organización cliente "Beta Ficticia S.A.S." con sus propios miembros
  Cuando ese usuario consulta la lista de miembros con su contexto de usuario propagado
  Entonces obtiene únicamente los miembros de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna fila de "Beta Ficticia S.A.S."
  Y un intento de escribir una membresía en "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

```gherkin
Escenario: Retirar a un miembro no borra lo que ya hizo
  Dado un usuario miembro de "Alfa Ficticia S.A.S." con acciones ya registradas en la bitácora
  Cuando se revoca su membresía
  Entonces pierde el acceso a los datos de esa organización cliente
  Y las entradas de bitácora que lo mencionan siguen existiendo sin alteración
  Y la revocación queda registrada en la bitácora con quién la ejecutó y cuándo
```

## Reglas de negocio

- Toda cuenta de usuario se identifica por un correo electrónico único en la plataforma.
- La membresía es la relación entre un usuario y una organización cliente; lleva exactamente
  un rol, y ese rol solo tiene efecto dentro de esa organización cliente.
- Un usuario puede tener varias membresías simultáneas en organizaciones clientes distintas.
- Una organización cliente no puede quedarse sin ningún miembro con rol de Administrador.
- La membresía se revoca, no se borra: revocar deja rastro; borrar dejaría un hueco en la
  bitácora.
- **Toda tabla de esta historia salvo la de cuentas de usuario lleva `organization_id`** y su
  política de aislamiento. La tabla de cuentas es la única del sustrato que vive por encima de
  las organizaciones clientes, precisamente porque un usuario puede pertenecer a varias.

## Fuera de alcance

- Pantallas: la Fase 0 no entrega interfaz. Se crean organizaciones y membresías por migración
  o por operación de sistema.
- Flujo de invitación por correo, aceptación y recuperación de contraseña.
- Definición y ajuste de los permisos de cada rol → `HU-003`.
- Segundo factor de autenticación, y el acceso de la contraparte por enlace.
- Planes, licencias y cupos → Fase C, `ADR-0002`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `organizacion.nombre` | Sí | Texto no vacío | No |
| `organizacion.identificacion_tributaria` | No | Formato colombiano `(por validar)`; único si viene | Sí |
| `organizacion.estado` | Sí | `activa` \| `suspendida` | No |
| `usuario.correo` | Sí | Correo válido, único en la plataforma | Sí (dato personal) |
| `usuario.nombre` | Sí | Texto no vacío | Sí (dato personal) |
| `membresia.organization_id` | Sí | Organización cliente existente | No |
| `membresia.usuario_id` | Sí | Cuenta existente | No |
| `membresia.rol_id` | Sí | Rol vigente de esa organización cliente (`HU-003`) | No |
| `membresia.estado` | Sí | `activa` \| `revocada` | No |
| `membresia` (unicidad) | Sí | Una sola membresía activa por par usuario–organización cliente | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §30 (roles), §31 (multiempresa)
- Decisiones: `ADR-0001` (identidad, organización desde el día uno)
- Contexto: `00-contexto/actores-y-roles.md`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-024` — si el cliente puede crear roles propios, `membresia.rol_id`
  apunta a una tabla configurable por organización cliente; si solo ajusta la matriz de los seis
  roles definidos, apunta a un catálogo común. **La historia queda en `borrador` hasta que se
  responda.**
- **Supuestos:** `SUP-005` (todo es una organización), `SUP-002` (multiempresa, confirmado §31).
- **Depende de:** nada. Es la primera de la épica.
- **Habilita a:** `HU-002`, `HU-003`, `HU-006`.
