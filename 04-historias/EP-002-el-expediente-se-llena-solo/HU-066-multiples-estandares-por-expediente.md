---
id: HU-066
titulo: Múltiples estándares sobre un mismo expediente
estado: borrador
epica: EP-002
prioridad: Should
actualizado: 2026-09-17
---

# HU-066 — Múltiples estándares sobre un mismo expediente

> **Nota de alcance (2026-09-17).** Esta historia nace de prueba en vivo, no del documento del
> cliente. Es, con diferencia, el cambio de mayor tamaño de este lote: toca cómo se indexa toda
> la matriz de requisitos (`HU-007`, `HU-035`), no solo la pantalla de creación del expediente
> (`HU-008`). **Se deja especificada aquí para no perder la intención, pero su implementación
> requiere su propia sesión de exploración y diseño técnico antes de dispatcharse** — no entra
> en el mismo lote de ajustes rápidos que el resto de hallazgos de esta ronda.

## Historia

**Como** Usuario operativo
**quiero** indicar más de un estándar aplicable al crear la solicitud de vinculación (por
ejemplo SARLAFT y PTEE a la vez)
**para** que un solo expediente exija todo lo que corresponde, sin tener que abrir un
expediente por cada estándar sobre la misma contraparte.

## Contexto

Hoy `dossier.standard` (`HU-008`) es un único valor por expediente, y la matriz de requisitos
(`HU-007`, glosario: *"la tabla configurable que define qué campo, documento y fuente aplica a
cada combinación de estándar y tipo de contraparte"*) ya está indexada por estándar × tipo de
contraparte — pero un expediente solo cita **uno** de esos estándares, nunca varios a la vez.

En la práctica, la misma contraparte casi siempre debe pasar por más de un marco a la vez (una
empresa vigilada por Supersociedades típicamente necesita SAGRILAFT y PTEE juntos, ver
`00-contexto/vision.md`). Hoy eso obliga a abrir un expediente por estándar sobre la misma
contraparte, lo que **rompe la promesa de `HU-008`** de que el sujeto es uno solo por
organización cliente y el expediente es "la unidad central" donde cuelga toda la evidencia:
con un estándar por expediente, la evidencia de la misma contraparte queda repartida en
expedientes distintos que no se comunican entre sí.

## Criterios de aceptación

```gherkin
Escenario: Crear una solicitud con más de un estándar
  Dado una versión de configuración vigente con SARLAFT y PTEE activos para el tipo "proveedor"
  Cuando un usuario operativo crea una solicitud para una contraparte de tipo "proveedor" seleccionando ambos estándares
  Entonces queda creado un único expediente que cita los dos estándares
  Y la creación queda registrada en la bitácora con los dos estándares indicados
```

```gherkin
Escenario: El expediente exige la unión de lo que pide cada estándar activo
  Dado un expediente abierto con dos estándares
  Cuando se consultan sus requisitos pendientes
  Entonces aparecen los campos y documentos que exige cada estándar para ese tipo de contraparte
  Y un requisito exigido por ambos estándares a la vez no aparece duplicado
```

```gherkin
Escenario: Cada estándar conserva su propia evaluación
  Dado un expediente con dos estándares y sus requisitos ya conciliados
  Cuando se evalúa el riesgo o se aplican las reglas de cada estándar
  Entonces cada estándar produce su propio resultado de evaluación
  Y un resultado de un estándar no se mezcla ni se promedia con el de otro
```

```gherkin
Escenario: La decisión puede diferir entre estándares del mismo expediente
  Dado un expediente con dos estándares evaluados
  Cuando el Oficial de Cumplimiento decide sobre el expediente
  Entonces puede aprobar respecto a un estándar y dejar pendiente o rechazar respecto al otro
  Y el estado general del expediente refleja el peor de los dos hasta que ambos queden resueltos
```

```gherkin
Escenario: Retirar un estándar de la organización no rompe expedientes abiertos
  Dado un expediente abierto citando un estándar que luego se retira de la configuración
  Cuando se publica esa configuración sin el estándar retirado
  Entonces el expediente ya abierto conserva ese estándar y sus requisitos, tal como los citó
  Y no se pueden abrir expedientes nuevos con el estándar retirado
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre expedientes multi-estándar
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta expedientes con más de un estándar con su contexto de usuario propagado
  Entonces obtiene únicamente los de expedientes de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Un expediente puede citar **uno o más** estándares de la versión de configuración con la que
  se abrió (`HU-008`); la cita no cambia después, igual que hoy con un solo estándar.
- Los requisitos pendientes del expediente son la **unión** de lo que exige la matriz para cada
  estándar citado y el tipo de contraparte, sin duplicar un requisito que coincide entre
  estándares.
- Cada estándar mantiene su **propia** evaluación de riesgo y su propio resultado (`ADR-0005`
  §3): no existe una evaluación combinada que mezcle reglas de estándares distintos.
- La decisión final (`HU-015`) puede resolver cada estándar de forma independiente; el estado
  visible del expediente refleja el estándar menos avanzado hasta que todos queden resueltos.
- Retirar un estándar de la configuración de la organización (`HU-035`) no afecta a los
  expedientes que ya lo citaban, solo impide abrir expedientes nuevos con él.

## Fuera de alcance

- El rediseño de cómo se indexa la matriz de requisitos en la base de datos: esta historia
  asume que la matriz sigue siendo estándar × tipo de contraparte (`HU-007`), y que "varios
  estándares en un expediente" es una unión de consultas a esa misma matriz, no una matriz
  nueva. Si el diseño técnico concluye que hace falta un modelo distinto, eso se decide en la
  sesión de diseño dedicada, no aquí.
- La migración de expedientes ya abiertos con un solo estándar hacia el modelo de varios.
- Cambiar la pantalla de administración de la matriz (`HU-035`) más allá de lo que ya hace hoy
  por estándar.
- Informes o consolidados que crucen resultados entre estándares de expedientes distintos.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `dossier.standards` | Sí | Uno o más valores de estándar de la versión de configuración citada; reemplaza a `dossier.standard` | No |
| `requirement.standard` | Sí | Sin cambios respecto a `HU-007`/`HU-035`: sigue siendo un valor por requisito | No |
| `evaluation.standard` | Sí | Toda evaluación (`ADR-0005` §3) cita a qué estándar del expediente pertenece | No |
| `decision.standard` | Sí | Toda decisión (`HU-015`) cita a qué estándar del expediente pertenece | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: §y visión de "estándares activables por organización cliente"
  (`AGENTS.md`, `00-contexto/vision.md`)
- Historias: extiende `HU-008` (creación), `HU-007`/`HU-035` (matriz), `HU-015` (decisión)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna formalizada todavía; conviene abrir una `PA-xxx` si el diseño
  técnico revela que `dossier.standards` como arreglo no alcanza (por ejemplo, si dos
  estándares exigen valores distintos para el mismo campo declarado).
- **Depende de:** `HU-007`, `HU-008`, `HU-035`.
- **Habilita a:** que un cliente con contrapartes sujetas a SARLAFT y PTEE a la vez (el caso más
  común según `00-contexto/vision.md`) no tenga que fragmentar su evidencia en expedientes
  separados.
- **Riesgo (el más alto de este lote):** toca el modelo de datos central de la matriz de
  requisitos y el motor de evaluación, no solo una pantalla. Implementarla sin antes dimensionar
  el impacto en `requirement-matrix.ts`, en el motor de evaluación de requisitos y en cómo
  hoy se asume "un estándar por expediente" en el código existente, puede introducir
  regresiones en `HU-007`, `HU-017` y `HU-019`. Por eso queda fuera del lote de planes rápidos
  de esta ronda.
