---
id: HU-029
titulo: Personas relacionadas y grafo de relaciones
estado: borrador
epica: EP-004
prioridad: Must
actualizado: 2026-08-27
---

# HU-029 — Personas relacionadas y grafo de relaciones

## Historia

**Como** Analista de Cumplimiento
**quiero** ver quién está detrás de una contraparte y qué diligencia exige cada uno según su tipo
de relación
**para** no tratar igual a un representante legal que a un accionista minoritario, ni pedirle a
todos lo mismo por si acaso.

## Contexto

Es la Fase 10, y empieza corrigiendo a la versión anterior del documento: **no** se crea
automáticamente una sub-solicitud por cada nombre que aparezca en un documento. Eso multiplica el
trabajo y el costo sin criterio.

En su lugar, el sistema construye un **grafo de relaciones**:

```
Empresa A
 ├─ representante legal → Persona 1
 ├─ accionista          → Empresa B
 ├─ accionista          → Persona 2
 ├─ beneficiario final  → Persona 3
 └─ apoderado           → Persona 4
```

Cada relación guarda tipo, fuente de donde salió, fecha, porcentaje de participación cuando
aplica, evidencia y estado de verificación. Y un **motor de relaciones** configurable decide qué
necesita cada quien: identificación, consulta a listas, revisión de PEP, documentación propia,
debida diligencia completa, intensificada, o simplemente quedar registrado.

`05-datos/modelo-conceptual.md` ya fijó la consecuencia técnica: **las relaciones son aristas de
primera clase**, no columnas en una tabla de personas. Es lo que permite el grafo y las consultas
recursivas.

## Criterios de aceptación

```gherkin
Escenario: Registrar una relación con su evidencia
  Dado un expediente sobre la contraparte "Ficticia S.A.S."
  Cuando se registra que una persona es su representante legal
  Entonces la relación queda guardada con su tipo, la fuente de donde salió, la fecha y su evidencia
  Y el sujeto relacionado queda registrado a nivel de la organización cliente
  Y la relación nace con estado de verificación pendiente
```

```gherkin
Escenario: Cada tipo de relación recibe el tratamiento que su configuración define
  Dado una configuración en la que el representante legal exige identificación y consulta a listas
  Y en la que el accionista minoritario solo exige quedar registrado
  Cuando se registran ambas relaciones en un expediente
  Entonces el sistema exige para cada una lo que su tipo define
  Y la diferencia proviene de la configuración, no de una condición escrita en el programa
```

```gherkin
Escenario: No se crea diligencia para todo nombre que aparezca
  Dado un documento del que se extraen varios nombres sin tipo de relación determinado
  Cuando se procesan
  Entonces quedan como candidatos a relación, pendientes de que una persona confirme el tipo
  Y no se abre ninguna diligencia ni se consulta ninguna fuente por ellos
```

```gherkin
Escenario: Recorrer el grafo hasta varios niveles
  Dado una contraparte cuya accionista es otra empresa, que a su vez tiene socios personas
  Cuando se consulta el grafo de relaciones del expediente
  Entonces se obtienen los dos niveles con sus tipos, porcentajes y evidencias
  Y se indica hasta qué profundidad se recorrió
```

```gherkin
Escenario: Un mismo sujeto en dos expedientes de la misma organización cliente
  Dado una persona ya registrada como representante legal en otro expediente de "Alfa Ficticia S.A.S."
  Cuando aparece de nuevo en un expediente nuevo de esa misma organización cliente
  Entonces el sistema lo señala como el mismo sujeto
  Y su información previa se puede reutilizar si la base jurídica y la vigencia lo permiten
  Y esa reutilización queda registrada
```

```gherkin
Escenario: Nunca se reutiliza entre organizaciones clientes
  Dado una persona registrada como relacionada en un expediente de "Alfa Ficticia S.A.S."
  Cuando "Beta Ficticia S.A.S." registra a esa misma persona
  Entonces son dos sujetos distintos, con evidencia independiente
  Y nada de lo aportado en una organización cliente se usa en la otra
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el grafo
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta relaciones con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
  Y un intento de escribir una relación en "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Una relación es una **entidad de primera clase** con: sujeto origen, sujeto destino, tipo,
  fuente, fecha, porcentaje de participación cuando aplique, evidencia y estado de verificación.
- **No se abre diligencia por cada nombre detectado.** Un nombre extraído es un candidato hasta
  que una persona confirme su tipo de relación.
- El **motor de relaciones** es configuración versionada (`ADR-0004`): asocia cada tipo de
  relación con el tratamiento que exige, dentro de un conjunto cerrado —registrar, identificar,
  consultar listas, revisar condición de PEP, exigir documentación propia, diligencia completa,
  diligencia intensificada.
- Los sujetos viven a nivel de **organización cliente** y se reutilizan entre expedientes de esa
  misma organización cuando la base jurídica y la vigencia lo permiten (§46). **Nunca entre
  clientes del SaaS** (§31).
- El origen de cada relación se conserva: una relación declarada por la contraparte no es lo
  mismo que una extraída de un documento ni que una verificada contra un registro público.
- La profundidad del recorrido del grafo es configurable y acotada: sin límite, un grafo
  societario puede crecer sin fin y el costo de consultas con él.

## Fuera de alcance

- La determinación del beneficiario final → `HU-030`.
- El cálculo de riesgo que usa la estructura societaria como factor → `HU-032`.
- El screening de las personas relacionadas: usa `HU-025` con el alcance que el motor de
  relaciones determine; aquí se decide **a quién** hay que consultar, no cómo.
- Grafos complejos, análisis societario avanzado y visualizaciones elaboradas → fase posterior
  (§36).
- La diligencia completa de una persona relacionada como expediente propio: es un expediente más
  y usa `EP-001`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `relacion.organization_id` | Sí | Organización cliente existente | No |
| `relacion.origen_sujeto_id` | Sí | Sujeto de la misma organización cliente | Sí |
| `relacion.destino_sujeto_id` | Sí | Sujeto de la misma organización cliente | Sí |
| `relacion.tipo` | Sí | Tipo definido en la versión de configuración | No |
| `relacion.fuente_dato` | Sí | `declarado` \| `extraído` \| `verificado` | No |
| `relacion.fecha` | No | Fecha desde la que rige la relación | No |
| `relacion.porcentaje` | Condicional | Obligatorio si el tipo lo requiere; entre 0 y 100 | Sí |
| `relacion.evidencia_id` | Condicional | Obligatoria si la fuente es extraída o verificada | Sí |
| `relacion.estado_verificacion` | Sí | `pendiente` \| `verificada` \| `no_verificable` | No |
| `tratamiento_relacion.tipo` | Sí | Conjunto cerrado del motor de relaciones | No |
| `profundidad_maxima` | Sí | Configurable; acota el recorrido del grafo | No |

## Trazabilidad

- Épica: `EP-004`
- Capacidad: `CAP-04`
- Documento del cliente: Fase 10, §31, §46, §37
- Decisiones: `ADR-0004` (motor de relaciones configurable), `ADR-0001` (consultas recursivas en
  Postgres), `05-datos/modelo-conceptual.md` (aristas de primera clase)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-018` (cuántos tipos de relación hay que soportar), `PA-017` (quién
  los configura). **Queda en `borrador`.**
- **Supuestos:** `SUP-002` (aislamiento).
- **Depende de:** `HU-008` (sujetos), `HU-012` y `HU-017` (de dónde salen las relaciones),
  `HU-031` (qué factores pondera la metodología).
- **Habilita a:** `HU-030`, `HU-032`, y el screening de relacionados.
- **Riesgo:** el grafo es donde el costo se multiplica sin avisar. Cada persona relacionada a la
  que se le exige consulta a listas es una consulta más que se paga, y la §31 impide compartirla
  entre clientes. La configuración del motor de relaciones es, en la práctica, una palanca de
  costo.
