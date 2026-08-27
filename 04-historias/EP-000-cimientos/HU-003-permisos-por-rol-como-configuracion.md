---
id: HU-003
titulo: Permisos por rol como configuración de la organización
estado: borrador
epica: EP-000
prioridad: Must
actualizado: 2026-08-27
---

# HU-003 — Permisos por rol como configuración de la organización

## Historia

**Como** Administrador de una organización cliente
**quiero** que la matriz de permisos de mis roles sea configuración de mi organización y no una
lista fija dentro del programa
**para** ajustar la segregación de funciones a mi política interna sin que haya que modificar y
desplegar el producto.

## Contexto

La §30 dice que la matriz de permisos es **un ejemplo base** y que "el cliente puede ajustar
según su propia segregación de funciones". `ADR-0004` traduce eso a una restricción dura: el
control de acceso es configuración versionada, **no un `enum` en el código**. El código conoce
la forma de un permiso; nunca la lista de quién lo tiene.

La distinción con `HU-002` importa y conviene fijarla aquí: el aislamiento decide **de qué
organización cliente** son las filas que se ven; el rol decide **qué se puede hacer** con
ellas. Son dos controles distintos y ninguno sustituye al otro.

La matriz base de la que se parte está en `00-contexto/actores-y-roles.md` y se carga como
datos, no como código.

## Criterios de aceptación

```gherkin
Escenario: Cargar la matriz base como datos de la organización cliente
  Dado una organización cliente recién creada
  Cuando se carga su configuración inicial de roles y permisos
  Entonces existen los roles Administrador, Oficial de Cumplimiento, Analista de Cumplimiento, Revisor, Auditor y Usuario operativo
  Y cada permiso de cada rol es una fila de configuración, no una constante del programa
  Y una búsqueda de la matriz de permisos en el código fuente no encuentra ninguna asignación literal de permisos a roles
```

```gherkin
Escenario: Ajustar la matriz sin desplegar el producto
  Dado la organización cliente "Alfa Ficticia S.A.S." cuyo rol de Analista de Cumplimiento no puede aprobar
  Cuando el Administrador publica una versión de configuración en la que ese rol sí puede aprobar
  Entonces los usuarios con ese rol en esa organización cliente pueden aprobar desde ese momento
  Y no fue necesario desplegar una versión nueva del programa
  Y el cambio queda registrado en la bitácora con quién lo publicó, cuándo y qué versión quedó vigente
```

```gherkin
Escenario: La matriz de una organización cliente no afecta a otra
  Dado que en "Alfa Ficticia S.A.S." el rol de Analista de Cumplimiento puede aprobar
  Y que en "Beta Ficticia S.A.S." ese mismo rol no puede aprobar
  Cuando un usuario con rol de Analista de Cumplimiento en ambas intenta aprobar en cada una
  Entonces la acción se permite en "Alfa Ficticia S.A.S."
  Y se rechaza en "Beta Ficticia S.A.S."
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las tablas de configuración de acceso
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta las tablas de roles y permisos con su contexto de usuario propagado
  Entonces obtiene únicamente las filas de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna fila de "Beta Ficticia S.A.S."
  Y un intento de modificar la matriz de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

```gherkin
Escenario: Una acción sin permiso se rechaza y queda registrada
  Dado un usuario cuyo rol vigente no incluye el permiso de aprobar
  Cuando intenta aprobar
  Entonces la acción es rechazada
  Y la bitácora registra el intento con el usuario, la acción pretendida y la versión de configuración con la que se evaluó
```

```gherkin
Escenario: El Auditor solo consulta
  Dado un usuario con rol de Auditor en "Alfa Ficticia S.A.S."
  Cuando consulta la bitácora de esa organización cliente
  Entonces obtiene el registro completo de la organización cliente
  Y cualquier intento suyo de escribir sobre datos del dominio es rechazado
```

## Reglas de negocio

- La matriz de permisos es **configuración versionada** (`ADR-0004`): no se edita en sitio, se
  publica una versión nueva mediante el mecanismo de `HU-004`. La versión anterior queda intacta.
- Ningún permiso, rol ni combinación de ambos aparece literalmente en el código. El código
  pregunta "¿este usuario tiene este permiso aquí y ahora?"; la respuesta sale de la
  configuración vigente de esa organización cliente.
- El conjunto de **permisos** posibles sí es cerrado y versionado —es la "forma" que el código
  conoce—; lo que es abierto es **su asignación a roles**.
- Toda evaluación de permiso registra con qué versión de configuración se resolvió, para que un
  rechazo o una autorización de hace un año se pueda reconstruir.
- Una organización cliente no puede publicar una versión que la deje sin ningún rol capaz de
  administrar la configuración.
- Las tablas de roles y de asignación de permisos llevan `organization_id` y su política de
  aislamiento.

## Fuera de alcance

- La interfaz de administración de roles: llega en la Fase 5. Hasta entonces la configuración
  se carga por operación de sistema.
- El catálogo de permisos del expediente, las alertas y el motor de riesgo: se irán ampliando
  cuando existan esos módulos. Aquí se construye el mecanismo y se carga la matriz base de la §30.
- La aprobación de una vinculación en sí misma (Fase 1): aquí solo existe el permiso, no la
  acción que gobierna.
- Delegaciones temporales, aprobación por doble control y jerarquías de rol.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `rol.organization_id` | Sí | Organización cliente existente | No |
| `rol.nombre` | Sí | Único dentro de la organización cliente y de la versión de configuración | No |
| `permiso.clave` | Sí | Debe existir en el catálogo cerrado de permisos de la versión | No |
| `asignacion.rol_id` | Sí | Rol de la misma versión de configuración | No |
| `asignacion.permiso_clave` | Sí | Permiso del catálogo | No |
| `asignacion.version_configuracion_id` | Sí | Versión de configuración existente (`HU-004`) | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §30
- Decisiones: `ADR-0004` (el control de acceso es configuración, no `enum`)
- Contexto: `00-contexto/actores-y-roles.md` (matriz base)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-024` — bloqueante. Si el cliente puede **crear roles propios**, el
  rol es una fila configurable por organización cliente; si solo puede **ajustar los permisos de
  los seis roles definidos**, el catálogo de roles es común y solo la asignación es
  configurable. Son dos modelos de datos distintos. **La historia no pasa de `borrador` hasta
  que se responda.** También queda pendiente quién asigna roles a los miembros.
  `PA-017` y `PA-018` afectan al esfuerzo, no al modelo.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-001` (roles y membresías), `HU-002` (aislamiento), `HU-004` (mecanismo de
  publicación versionada).
- **Habilita a:** toda acción sujeta a permiso en las fases siguientes.
