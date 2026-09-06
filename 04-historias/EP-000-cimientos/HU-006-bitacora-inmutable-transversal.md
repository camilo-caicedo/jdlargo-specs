---
id: HU-006
titulo: Bitácora inmutable transversal
estado: implementado
epica: EP-000
prioridad: Must
actualizado: 2026-09-06
---

# HU-006 — Bitácora inmutable transversal

> **Actualización 2026-09-05 (`PA-026`, `PA-009`).** El cliente confirmó que basta con
> **solo inserción por permisos de base de datos**: no se exige encadenamiento criptográfico ni
> WORM. Pero se añade **hash por evento desde el día uno**, porque es lo que convierte el
> endurecimiento posterior en una migración y no en una reescritura. La retención es
> **configurable por organización cliente**, con 10 años como referencia `(por validar)` y copias
> periódicas al contacto que designe el cliente. Ver `ADR-0007`, `RNF-006` a `RNF-013`.

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
- Cada entrada registra como mínimo, según la §23: actor, tipo de actor (`user`, `system` o
  —desde la Fase 1— `counterparty`), acción, entidad y fila afectadas, momento, origen de la
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
| `audit_log.organization_id` | Sí | Organización cliente existente | No |
| `audit_log.actor_id` | Condicional | Obligatorio si el tipo de actor es `user` | No |
| `audit_log.actor_type` | Sí | `user` \| `system` \| `counterparty` (este último desde la Fase 1) | No |
| `audit_log.action` | Sí | Valor del catálogo cerrado de acciones | No |
| `audit_log.entity` y `audit_log.entity_id` | Sí | Identifican la fila afectada | No |
| `audit_log.occurred_at` | Sí | Momento; se escribe una sola vez | No |
| `audit_log.request_origin` | Sí | Dirección de red y agente desde donde se actuó | Sí (dato personal indirecto) |
| `audit_log.previous_value` / `audit_log.new_value` | Condicional | Obligatorios en toda modificación | Sí |
| `audit_log.reason` | Condicional | Obligatorio en las acciones que la configuración exige justificar | No |
| `audit_log.source` | No | Fuente externa que originó el cambio, si la hubo | No |
| `audit_log.automatic` | Sí | Verdadero o falso | No |
| `audit_log.ai_model` | No | Modelo, proveedor y versión, si intervino IA (§32) | No |
| `audit_log.configuration_version_id` | Sí | Versión vigente en ese momento (`HU-004`) | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §23 (Fase 23), §31, §32, §44
- Decisiones: `ADR-0005` (regla 5: una sola bitácora inmutable, transversal)
- Requisitos: alimenta la categoría de auditoría y trazabilidad de
  `03-requisitos/no-funcionales.md`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna. `PA-026` y `PA-009` **resueltas** → `ADR-0007`. Queda la
  derivada `PA-044` (quién asume la retención a 10 años), que no afecta la estructura.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-001` (hay un actor que registrar), `HU-002` (aislamiento).
- **Habilita a:** `HU-003`, `HU-004`, `HU-005` y todas las fases siguientes. Es la primera que
  conviene construir, aunque sea la última que se lee.
- **Riesgo:** el tramo sin bitácora es irrecuperable. Cualquier dato escrito antes de que esta
  historia esté terminada queda sin explicación para siempre.
