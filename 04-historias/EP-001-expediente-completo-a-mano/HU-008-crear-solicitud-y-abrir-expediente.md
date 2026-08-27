---
id: HU-008
titulo: Crear la solicitud de vinculación y abrir el expediente
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-008 — Crear la solicitud de vinculación y abrir el expediente

## Historia

**Como** Usuario operativo
**quiero** crear una solicitud de vinculación indicando a quién voy a vincular y bajo qué tipo
de contraparte
**para** que el proceso de debida diligencia arranque en la plataforma y deje de vivir en
correos y hojas de cálculo.

## Contexto

Es la Fase 3 del documento del cliente y el punto donde nace el expediente, que es la unidad
central del sistema. El usuario indica contraparte, tipo, estándar aplicable, responsable
interno y fecha límite; el sistema genera el identificador, el expediente, el enlace de acceso
y el estado inicial.

Una decisión que se toma aquí y condiciona todo lo demás: **el expediente cita la versión de
configuración con la que se abre**. Es lo que después permite explicar por qué se pidió lo que
se pidió, aunque la matriz haya cambiado tres veces desde entonces (§41).

Aquí también nace el **sujeto**: la contraparte como registro de la organización cliente. Vive
por encima del expediente porque el mismo proveedor puede tener varias vinculaciones con el
mismo cliente. Nunca se comparte entre clientes del SaaS (§31).

## Criterios de aceptación

```gherkin
Escenario: Crear una solicitud y abrir su expediente
  Dado un usuario con permiso de crear solicitudes en "Alfa Ficticia S.A.S."
  Y una versión de configuración vigente con el tipo de contraparte "proveedor"
  Cuando crea una solicitud para la contraparte "Ficticia S.A.S." de tipo "proveedor"
  Entonces queda creado un expediente con identificador único dentro de la organización cliente
  Y el expediente queda en el estado inicial de la máquina de estados
  Y el expediente cita la versión de configuración con la que se abrió
  Y queda registrado el responsable interno y la fecha límite
  Y la creación queda registrada en la bitácora
```

```gherkin
Escenario: El expediente hereda lo que exige la matriz
  Dado una versión de configuración cuya matriz exige seis campos y tres tipos documentales al tipo "proveedor"
  Cuando se abre un expediente de ese tipo
  Entonces el expediente conoce esos seis campos y esos tres tipos documentales como requisitos pendientes
  Y esos requisitos provienen de la versión citada, no de la configuración vigente en el futuro
```

```gherkin
Escenario: Publicar una versión nueva no altera un expediente ya abierto
  Dado un expediente abierto con la versión 3 de configuración
  Cuando el Oficial de Cumplimiento publica la versión 4 con requisitos distintos
  Entonces el expediente abierto sigue exigiendo lo que decía la versión 3
  Y los expedientes que se abran desde ese momento exigen lo que dice la versión 4
```

```gherkin
Escenario: No se abre un expediente sin tipo de contraparte válido
  Dado una organización cliente sin ninguna versión de configuración publicada
  Cuando un usuario intenta crear una solicitud de vinculación
  Entonces la operación es rechazada indicando que no hay configuración vigente
  Y no queda ningún expediente a medias
```

```gherkin
Escenario: Dos organizaciones clientes vinculan a la misma contraparte
  Dado que "Alfa Ficticia S.A.S." abre un expediente sobre la contraparte "Ficticia S.A.S."
  Y que "Beta Ficticia S.A.S." abre un expediente sobre esa misma contraparte
  Entonces existen dos sujetos distintos y dos expedientes independientes
  Y ninguna de las dos organizaciones clientes puede ver el expediente de la otra
  Y nada de lo aportado en uno se reutiliza en el otro
```

```gherkin
Escenario: Crear una solicitud exige permiso
  Dado un usuario cuyo rol vigente no incluye el permiso de crear solicitudes
  Cuando intenta crear una
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

## Reglas de negocio

- Una solicitud de vinculación abre **un** expediente. El expediente es la unidad central: toda
  la evidencia posterior cuelga de él.
- El expediente **cita la versión de configuración** con la que se abrió, y esa cita no cambia
  aunque después se publiquen otras versiones. `PA-025` define si la cita se fija al abrir o al
  evaluar.
- Los requisitos del expediente se derivan de la matriz de la versión citada (`HU-007`), no se
  copian a mano ni se escriben en el código.
- El sujeto sobre el que trata el expediente vive a nivel de organización cliente y **nunca se
  comparte entre clientes del SaaS** (§31). Dos organizaciones clientes que conozcan a la misma
  contraparte tienen dos sujetos distintos.
- El identificador del expediente es único dentro de la organización cliente y no se reutiliza.
- Crear una solicitud requiere permiso explícito (`HU-003`). Según la matriz base lo tienen el
  usuario operativo, el analista y el Oficial de Cumplimiento.
- La contraparte declarada al crear la solicitud es un dato **declarado por el usuario
  operativo**, no un hecho verificado: se registra como afirmación con ese origen (`HU-005`).

## Fuera de alcance

- La generación y entrega del enlace de acceso → `HU-010`.
- El formulario que la contraparte diligencia → `HU-012`.
- La reutilización de un sujeto ya conocido en otro expediente del mismo cliente (§46) → Fase 4,
  con el motor de relaciones.
- Vincular la solicitud a una operación comercial concreta: el documento lo menciona como
  opcional y no hay definición todavía.
- Carga masiva de solicitudes.
- La verificación de que la contraparte declarada existe realmente → Fase 3.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `expediente.organization_id` | Sí | Organización cliente existente | No |
| `expediente.codigo` | Sí | Único dentro de la organización cliente; no se reutiliza | No |
| `expediente.sujeto_id` | Sí | Sujeto de la misma organización cliente | Sí |
| `expediente.tipo_contraparte_id` | Sí | Tipo de la versión de configuración citada | No |
| `expediente.estandar` | Sí | Estándar declarado en esa versión | No |
| `expediente.version_configuracion_id` | Sí | Versión publicada (`HU-004`); no cambia después | No |
| `expediente.responsable_interno_id` | Sí | Usuario miembro activo de la organización cliente | No |
| `expediente.fecha_limite` | No | Fecha futura | No |
| `expediente.estado` | Sí | Estado válido de la máquina de estados (`HU-009`) | No |
| `sujeto.identificacion` | Sí | Tipo y número de documento; único por organización cliente | Sí (dato personal) |
| `sujeto.nombre_declarado` | Sí | Texto no vacío; se guarda como afirmación de origen `declarado` | Sí (dato personal) |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 3, §31, §37, §41, §44 pregunta A
- Decisiones: `ADR-0004` (versión citada), `ADR-0005` (la contraparte declarada es una
  afirmación)
- Modelo: `05-datos/modelo-conceptual.md`, planos de Sujetos y Expediente

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-025` — bloqueante. Define si el expediente congela la versión de
  configuración **al abrirse** o **al evaluarse**, y de ahí depende el escenario de "publicar una
  versión nueva no altera un expediente ya abierto". `PA-031` — quién entrega el enlace a la
  contraparte, que determina si crear la solicitud dispara un correo. **Queda en `borrador`.**
- **Supuestos:** `SUP-002` (aislamiento, confirmado §31).
- **Depende de:** `HU-004`, `HU-007` (matriz), `HU-009` (estado inicial), `HU-005`
  (afirmaciones), `HU-002`, `HU-006`.
- **Habilita a:** `HU-010` en adelante. Es la puerta de entrada del recorrido.
