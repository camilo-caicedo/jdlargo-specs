---
id: HU-005
titulo: Registro de afirmaciones con procedencia
estado: en-revision
epica: EP-000
prioridad: Must
actualizado: 2026-09-06
---

# HU-005 — Registro de afirmaciones con procedencia

> **Actualización 2026-09-05 (`PA-027`).** No hay precedencia universal. La escala por defecto
> es `verificado por fuente independiente › documental validado › declarado › extraído por IA sin
> validar`, y es **configurable por organización cliente**. Dos reglas que no se negocian: frente
> a la IA **manda lo declarado**, y la precedencia **nunca borra el origen** ni las afirmaciones
> desplazadas. Cada afirmación se persiste como `campo = valor + fuente + fecha + confianza +
> estado`. Ver `ADR-0005` §2.

## Historia

**Como** Sistema
**quiero** guardar cada dato del expediente como una afirmación con su origen, su autor y su
evidencia, y no poder sobrescribir ninguna
**para** que sea imposible que un dato pierda de dónde vino o que una fuente borre en silencio
lo que dijo otra.

## Contexto

La §2 es, en palabras del propio cliente, "la idea más importante para entender el resto":
recolección, extracción, verificación, evaluación y decisión **nunca deben mezclarse ni
presentarse como si fueran lo mismo**. Que la IA haya leído que alguien es beneficiario final
no significa que esté verificado.

`ADR-0005` traduce eso a una restricción de modelo de datos, no de pantalla: una tabla con una
columna `nombre` no puede cumplirlo, porque en el momento de escribir el valor se perdió su
procedencia. **El expediente no guarda valores: guarda afirmaciones.**

Un mismo campo puede tener varias afirmaciones simultáneas y contradictorias. Eso no es un
error: es el estado normal de un expediente en curso. El ejemplo del cliente:

```
razon_social · "Ficticia S.A.S."            · declarado  · contraparte        · —
razon_social · "Ficticia Logistica S.A.S."  · verificado · Cámara de Comercio · doc#0001
→ discrepancia abierta, revisión humana requerida
```

Esta historia construye la estructura y sus invariantes. En la Fase 0 solo existirá el origen
`declared`, así que todavía no hay contradicción posible: lo que se garantiza aquí es que
cuando lleguen la extracción y la verificación, el modelo ya las admita sin reescribirse.

## Criterios de aceptación

```gherkin
Escenario: Registrar una afirmación declarada por la contraparte
  Dado un expediente en curso de "Alfa Ficticia S.A.S."
  Cuando se registra el campo razon_social con el valor "Ficticia S.A.S."
  Entonces la afirmación queda guardada con origen declarado
  Y con quién la produjo, en qué momento y con qué evidencia asociada
  Y la afirmación queda registrada en la bitácora
```

```gherkin
Escenario: Una afirmación nunca se sobrescribe
  Dado una afirmación ya registrada para el campo razon_social
  Cuando se intenta modificar su valor o su origen
  Entonces la operación es rechazada por la base de datos
  Y la afirmación original permanece intacta
```

```gherkin
Escenario: Un dato nuevo se añade, no reemplaza
  Dado una afirmación de razon_social con valor "Ficticia S.A.S." y origen declarado
  Cuando se registra para el mismo campo el valor "Ficticia Logistica S.A.S." con origen verificado
  Entonces existen dos afirmaciones simultáneas para ese campo
  Y ambas conservan su origen, su autor, su momento y su evidencia
  Y ninguna de las dos queda marcada como falsa por la sola existencia de la otra
```

```gherkin
Escenario: No se puede registrar una afirmación sin origen
  Dado un expediente en curso
  Cuando se intenta registrar un dato sin indicar su origen
  Entonces la operación es rechazada
  Y no queda ninguna fila escrita
```

```gherkin
Escenario: Lo extraído por la IA no asciende solo a verificado
  Dado una afirmación con origen extraído y su nivel de confianza registrado
  Cuando no ha intervenido ni una persona autorizada ni una fuente externa
  Entonces esa afirmación conserva el origen extraído
  Y no existe ninguna operación que cambie su origen a verificado
  Y una afirmación verificada solo puede crearse citando la fuente externa o la persona que la respalda
```

```gherkin
Escenario: Resolver una discrepancia deja registro y no borra nada
  Dado dos afirmaciones contradictorias sobre el mismo campo
  Cuando una persona autorizada resuelve la discrepancia eligiendo una de ellas
  Entonces se registra la resolución con quién decidió, cuándo y con qué fundamento
  Y la afirmación no elegida sigue existiendo como evidencia de que hubo una diferencia
  Y la resolución queda registrada en la bitácora
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las afirmaciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta las afirmaciones con su contexto de usuario propagado
  Entonces obtiene únicamente las afirmaciones de expedientes de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna afirmación de "Beta Ficticia S.A.S."
  Y un intento de registrar una afirmación en un expediente de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Toda afirmación guarda: organización cliente, expediente, sujeto, campo, valor, **origen**,
  quién o qué la produjo, cuándo, evidencia asociada, nivel de confianza si vino de un modelo, y
  estado.
- Los orígenes son cuatro y forman un conjunto cerrado: `declared`, `extracted`, `verified`,
  `evaluated`. **La decisión no es un origen**: es un evento aparte, con persona identificada,
  cargo y fundamento.
- La tabla de afirmaciones es de **solo inserción**. Corregir es añadir una afirmación nueva;
  nunca modificar la anterior.
- Un mismo campo admite varias afirmaciones vigentes a la vez, incluso contradictorias.
- Una afirmación de origen `extracted` **no puede** ascender a `verified`. Lo verificado se
  crea, citando la fuente externa o la persona que lo respalda; no se asciende.
- Toda afirmación de origen `extracted` registra el modelo, el proveedor y la confianza que
  reportó, y una afirmación producida por la IA **no puede quedar sin constancia de que la
  produjo ella** (§32).
- Ningún camino del sistema escribe un dato del expediente por fuera de esta estructura. Un
  valor plano en una columna del expediente es un defecto, no una optimización.

## Fuera de alcance

- **El cálculo del valor vigente y la regla de precedencia entre orígenes.** Es un cálculo, no
  una columna (`ADR-0005`), y su regla es configurable → `PA-027`, se implementa en la Fase 2
  junto con la conciliación.
- La extracción con IA y la verificación externa: aquí se admiten esos orígenes, no se producen.
- La pantalla de conciliación y la comparación declarado contra extraído → Fase 2.
- Las vistas materializadas de valor vigente. Son una optimización, y nunca autoritativas.
- El expediente y el sujeto en sí: se crean en la Fase 1. En la Fase 0 se construye la
  estructura de la afirmación y se verifican sus invariantes.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `assertion.organization_id` | Sí | Organización cliente existente | No |
| `assertion.dossier_id` | Sí | Expediente de la misma organización cliente (Fase 1) | No |
| `assertion.party_id` | Sí | Sujeto del expediente (Fase 1) | Sí |
| `assertion.field` | Sí | Campo definido en la versión de configuración aplicable | No |
| `assertion.value` | Sí | Según el tipo del campo en la configuración | Sí (puede ser dato personal) |
| `assertion.origin` | Sí | `declared` \| `extracted` \| `verified` \| `evaluated` | No |
| `assertion.produced_by` | Sí | Usuario, contraparte, fuente externa o ejecución de IA | No |
| `assertion.produced_at` | Sí | Momento; se escribe una sola vez | No |
| `assertion.evidence_id` | Condicional | Obligatoria si el origen es `extracted` o `verified` | Sí |
| `assertion.confidence` | Condicional | Obligatoria si el origen es `extracted`; entre 0 y 1 | No |
| `assertion.configuration_version_id` | Sí | Versión con la que se registró (`HU-004`) | No |
| `assertion.status` | Sí | `active` \| `discarded` — descartar no borra | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §2, §32, §37, §44
- Decisiones: `ADR-0005`
- Modelo: `05-datos/modelo-conceptual.md`, plano de Expediente

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna. `PA-027` **resuelta** → `ADR-0005` §2.
- **Supuestos:** `SUP-006` (la IA propone y justifica; la decisión la firma una persona).
- **Depende de:** `HU-002` (aislamiento), `HU-004` (versión de configuración que se cita),
  `HU-006` (bitácora).
- **Habilita a:** el expediente completo de la Fase 1, la conciliación de la Fase 2, la
  verificación de la Fase 3 y el monitoreo continuo de la Fase 6.
- **Riesgo:** es la historia donde más se equivoca quien llega con costumbres de CRUD. La
  tentación de "guardar el valor final y ya" reaparece en cada formulario, y cada vez que se
  cede se pierde una discrepancia que después nadie puede reconstruir.
