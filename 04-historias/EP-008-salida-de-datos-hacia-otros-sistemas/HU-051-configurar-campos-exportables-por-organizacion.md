---
id: HU-051
titulo: Configurar qué campos del expediente son exportables por organización
estado: borrador
epica: EP-008
prioridad: Must
actualizado: 2026-09-06
---

# HU-051 — Configurar qué campos del expediente son exportables por organización

## Historia

**Como** Administrador de una organización cliente
**quiero** decidir qué campos del expediente pueden salir hacia otro sistema, y que ciertos
campos queden bloqueados sin importar lo que yo configure
**para** controlar qué información sale de la plataforma sin arriesgar datos personales que no
debían salir.

## Contexto

`PA-010` confirma que la plataforma alimenta otros sistemas del cliente, no los reemplaza. Antes
de construir cualquier medio de salida (archivo o interfaz de programación, `HU-052` y `HU-053`),
hace falta decidir **qué** puede salir: la lista de campos exportables no puede ser fija en el
programa, porque cada organización cliente maneja información distinta y con distinta
sensibilidad.

Esta historia sigue el mismo patrón que el resto de la configuración de cumplimiento
(`ADR-0004`): la lista de campos exportables es contenido de una versión de configuración, no
código. Y hereda una restricción que no depende de lo que decida el cliente: hay campos que la
propia plataforma bloquea, por ser datos personales sensibles o de terceros, independientemente
de lo que la organización configure.

## Criterios de aceptación

```gherkin
Escenario: Marcar un campo como exportable
  Dado una versión de configuración en borrador de una organización cliente
  Cuando el Administrador marca un campo del expediente como exportable
  Y publica la versión
  Entonces ese campo queda disponible para salir por archivo o por interfaz de programación
```

```gherkin
Escenario: Ningún campo es exportable por defecto
  Dado una organización cliente que nunca configuró campos exportables
  Cuando se intenta exportar cualquier expediente suyo
  Entonces no sale ningún campo
  Y el sistema no asume una lista de campos por defecto
```

```gherkin
Escenario: Un campo bloqueado por política de plataforma no se puede marcar exportable
  Dado un campo marcado como bloqueado por política de plataforma
  Cuando el Administrador intenta marcarlo como exportable
  Entonces el sistema rechaza la operación
  Y el campo permanece no exportable sin excepción posible desde la configuración del cliente
```

```gherkin
Escenario: Cambiar la configuración no reescribe extracciones ya hechas
  Dado un campo que era exportable cuando se generó una extracción anterior
  Cuando el Administrador retira ese campo de la lista de exportables
  Entonces las extracciones ya generadas no se alteran ni se revocan
  Y solo las extracciones futuras dejan de incluir ese campo
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre esta configuración
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta o modifica la lista de campos exportables con su contexto de usuario propagado
  Entonces obtiene y modifica únicamente la de su propia organización
  Y un intento de modificar la de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- La lista de campos exportables es **configuración versionada por organización cliente**
  (`ADR-0004`), nunca una lista fija en el programa.
- El valor por defecto es **no exportable**: un campo solo sale si alguien lo marcó
  explícitamente. No hay opt-out, solo opt-in.
- Hay campos que la plataforma bloquea sin importar la configuración del cliente, por tratarse de
  datos personales sensibles o de terceros. **Cuáles exactamente es `TBD` — `PA-048`.**
- Cambiar esta configuración no es retroactivo: no reescribe ni invalida extracciones que ya
  salieron con la configuración anterior.

## Fuera de alcance

- La extracción en sí, por archivo o por interfaz de programación → `HU-052`, `HU-053`.
- La lista concreta de campos que la plataforma bloquea por política propia — depende de
  `PA-048`.
- Una interfaz gráfica de administración dedicada: por ahora esta configuración se administra
  igual que el resto de la configuración de cumplimiento (Fase 5, `EP-005`); no es una pantalla
  aparte.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `exportable_field.organization_id` | Sí | Organización cliente existente | No |
| `exportable_field.configuration_version_id` | Sí | Versión existente (`HU-004`) | No |
| `exportable_field.field` | Sí | Referencia a un campo del diccionario de datos (`05-datos/diccionario-de-datos.md`) | No |
| `exportable_field.enabled` | Sí | Booleano; por defecto `false` | No |
| `exportable_field.blocked_by_policy` | Sí | Booleano; solo lo fija el sistema, no el cliente `(TBD — PA-048)` | No |

## Trazabilidad

- Épica: `EP-008`
- Capacidad: `CAP-08`
- Decisiones: `ADR-0004` (configuración versionada, no código)
- Preguntas: cierra parcialmente la implementación de `PA-010`

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-048`** — qué campos quedan bloqueados por política de
  plataforma. Sin esto, la lista de "bloqueados por defecto" queda vacía y el control recae
  entero en lo que configure cada cliente.
- **Depende de:** `HU-004` (versiones de configuración), `HU-002` (aislamiento).
- **Habilita a:** `HU-052`, `HU-053`.
- **Riesgo:** marcar un campo como exportable sin haber cerrado `PA-048` puede dejar salir un
  dato que la ley no permite entregar a un tercero sin base jurídica. Mientras la pregunta siga
  abierta, el criterio por defecto es el más restrictivo: nada sale si no se marcó a mano.
