---
id: HU-006
titulo: Bitácora inmutable transversal
estado: borrador
epica: EP-000
prioridad: Must
actualizado: 2026-08-27
---

# HU-006 — Bitácora inmutable transversal

## Historia

**Como** Auditor
**quiero** que toda acción sobre datos de mi organización cliente quede registrada en una
bitácora que nadie pueda alterar ni borrar
**para** poder responder quién hizo qué, cuándo y bajo qué versión de regla, sin depender de que
alguien haya construido un reporte a propósito.

## Contexto

La §23 lo dice y `ADR-0005` lo repite: la bitácora **no es un módulo, es el sustrato**. Debe
registrar siempre quién, qué, cuándo, desde dónde, el valor anterior y el nuevo, el motivo del
cambio, la fuente, si fue un proceso automático o manual, qué modelo de IA intervino y qué
versión de regla estaba vigente en ese momento.

Se construye en la Fase 0, antes que nada, por una razón concreta: **una bitácora que empieza
tarde deja un tramo del historial sin explicación**, y ese tramo no se puede reconstruir después.

La §23 pide atención especial a: cambios en el nivel de riesgo, cambios de beneficiario final,
decisiones tomadas, eliminación o reemplazo de documentos, resolución de alertas y cualquier
anulación manual de un resultado del sistema. Ninguno de esos hechos existe todavía en la Fase
0; lo que se construye aquí es la estructura que los aceptará sin modificarse.

## Criterios de aceptación

```gherkin
Escenario: Toda escritura sobre el dominio deja rastro
  Dado un usuario autenticado y actuando en "Alfa Ficticia S.A.S."
  Cuando crea, modifica o revoca cualquier fila del dominio
  Entonces se escribe una entrada de bitácora con el actor, la acción, la tabla y la fila afectadas
  Y con el momento, el origen de la petición, el valor anterior y el valor nuevo
  Y con la versión de configuración vigente en ese momento
```

```gherkin
Escenario: La bitácora no se puede alterar
  Dado una entrada de bitácora ya escrita
  Cuando se intenta modificarla o eliminarla, incluso desde la conexión de administrador
  Entonces la operación es rechazada
  Y la entrada permanece idéntica
  Y el intento queda registrado como un hecho más de la bitácora
```

```gherkin
Escenario: Un proceso automático se distingue de una persona
  Dado un trabajo programado del sistema que modifica datos
  Cuando escribe su entrada de bitácora
  Entonces la entrada indica que el actor fue un proceso automático y cuál
  Y no queda atribuida a ninguna persona
  Y se distingue sin ambigüedad de la misma acción hecha a mano
```

```gherkin
Escenario: Una acción sin motivo no se registra a medias
  Dado una acción que la configuración exige justificar
  Cuando se ejecuta sin motivo
  Entonces la acción es rechazada por completo
  Y no queda ni el cambio de datos ni una entrada de bitácora parcial
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la bitácora
  Dado un usuario con rol de Auditor en "Alfa Ficticia S.A.S."
  Cuando consulta la bitácora con su contexto de usuario propagado
  Entonces obtiene únicamente las entradas de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna entrada de "Beta Ficticia S.A.S."
  Y ningún usuario puede escribir una entrada de bitácora en una organización cliente de la que no es miembro
```

```gherkin
Escenario: Reconstruir la historia de una fila
  Dado una fila del dominio que ha cambiado varias veces
  Cuando se pide su historial completo
  Entonces se obtiene la secuencia ordenada de todos sus cambios
  Y cada uno indica quién lo hizo, cuándo, desde dónde, qué valor tenía antes y qué valor quedó
  Y la secuencia no tiene huecos desde la creación de la fila
```

```gherkin
Escenario: El registro sobrevive a la baja del usuario
  Dado un usuario cuya membresía fue revocada
  Cuando se consulta la bitácora de sus acciones anteriores
  Entonces sus entradas siguen existiendo, completas e identificando quién era
  Y la revocación de la membresía aparece a su vez como una entrada más
```

## Reglas de negocio

- La bitácora es de **solo inserción**. No existe operación de modificación ni de borrado, y la
  restricción se impone en la base de datos: ni siquiera la conexión de administrador la evita.
- Cada entrada registra como mínimo, según la §23: actor, tipo de actor (`usuario`, `sistema` o
  —desde la Fase 1— `contraparte`), acción, entidad y fila afectadas, momento, origen de la
  petición, valor anterior, valor nuevo, motivo, fuente, si fue automático o manual, qué modelo
  de IA intervino y qué versión de configuración regía.
- La entrada de bitácora se escribe **en la misma transacción** que el cambio que describe. Si
  falla una, no ocurre la otra: no puede haber cambio sin rastro ni rastro sin cambio.
- La bitácora lleva `organization_id` y su política de aislamiento. Cada organización cliente
  tiene su propia bitácora (§31).
- Un intento rechazado —por permiso o por aislamiento— también se registra. Saber qué se intentó
  y no se pudo es parte de la auditoría.
- La bitácora no se depura. La política de retención se define cuando se responda `PA-009`, y
  hasta entonces no se borra nada.
- Los campos de modelo de IA quedan previstos desde ahora aunque no haya IA hasta la Fase 2:
  añadirlos después obligaría a dejar sin ellos todo lo ya registrado.

## Fuera de alcance

- La pantalla de consulta de auditoría y la exportación de reportes → Fase 6, panel del Oficial
  de Cumplimiento.
- El registro detallado de ejecuciones de IA como entidad propia (§32) → Fase 2. Aquí solo se
  reservan los campos en la entrada de bitácora.
- Los eventos de monitoreo continuo y de renovación → Fase 6.
- Alertas automáticas ante actividad sospechosa sobre la propia bitácora.
- La política de retención y purga, que depende de `PA-009`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `bitacora.organization_id` | Sí | Organización cliente existente | No |
| `bitacora.actor_id` | Condicional | Obligatorio si el tipo de actor es `usuario` | No |
| `bitacora.actor_tipo` | Sí | `usuario` \| `sistema` \| `contraparte` (este último desde la Fase 1) | No |
| `bitacora.accion` | Sí | Valor del catálogo cerrado de acciones | No |
| `bitacora.entidad` y `bitacora.entidad_id` | Sí | Identifican la fila afectada | No |
| `bitacora.ocurrido_en` | Sí | Momento; se escribe una sola vez | No |
| `bitacora.origen_peticion` | Sí | Dirección de red y agente desde donde se actuó | Sí (dato personal indirecto) |
| `bitacora.valor_anterior` / `valor_nuevo` | Condicional | Obligatorios en toda modificación | Sí |
| `bitacora.motivo` | Condicional | Obligatorio en las acciones que la configuración exige justificar | No |
| `bitacora.fuente` | No | Fuente externa que originó el cambio, si la hubo | No |
| `bitacora.automatico` | Sí | Verdadero o falso | No |
| `bitacora.modelo_ia` | No | Modelo, proveedor y versión, si intervino IA (§32) | No |
| `bitacora.version_configuracion_id` | Sí | Versión vigente en ese momento (`HU-004`) | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §23 (Fase 23), §31, §32, §44
- Decisiones: `ADR-0005` (regla 5: una sola bitácora inmutable, transversal)
- Requisitos: alimenta la categoría de auditoría y trazabilidad de
  `03-requisitos/no-funcionales.md`

## Dependencias y riesgos

- **Preguntas abiertas:**
  - `PA-026` — bloqueante. La §23 admite "inmutable **o** técnicamente protegido". Si basta con
    solo inserción por permisos de base de datos, la bitácora es una tabla más; si se exige
    encadenamiento criptográfico o almacenamiento WORM, cambia la estructura y el costo. **No
    pasa de `borrador` hasta que se responda**, porque rehacer la bitácora obliga a rehacer todo
    lo que ya escribió en ella.
  - `PA-009` — retención y trazabilidad. No bloquea la estructura, sí la política de conservación.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-001` (hay un actor que registrar), `HU-002` (aislamiento).
- **Habilita a:** `HU-003`, `HU-004`, `HU-005` y todas las fases siguientes. Es la primera que
  conviene construir, aunque sea la última que se lee.
- **Riesgo:** el tramo sin bitácora es irrecuperable. Cualquier dato escrito antes de que esta
  historia esté terminada queda sin explicación para siempre.
