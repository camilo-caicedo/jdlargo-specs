---
id: HU-026
titulo: Comparación de nombres e identificadores
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-026 — Comparación de nombres e identificadores

## Historia

**Como** Analista de Cumplimiento
**quiero** revisar cada parecido que encontró el sistema comparando identificadores, y dejar
registrado si lo descarto o lo confirmo
**para** que nadie quede señalado por llamarse parecido a otra persona, y para poder explicar por
qué decidí lo que decidí.

## Contexto

Es la §12.3 y la §12.4, y contiene el punto que el documento del cliente marca como clave y que
la versión anterior no tenía:

> Una coincidencia técnica **no significa que la persona esté sancionada**.

El flujo correcto es explícito:

```
Coincidencia técnica (la encontró el sistema) → revisión humana
→ comparación de identificadores → se descarta | probable | confirmada → decisión
```

Los cinco estados posibles son `sin coincidencia`, `posible coincidencia`, `coincidencia
descartada`, `coincidencia confirmada` y `pendiente de revisión`. La revisión humana de cada una
**queda siempre documentada**.

La comparación debe soportar coincidencia exacta, por similitud, alias, nombres invertidos,
errores de digitación, homónimos, transliteraciones e identificadores como documento de
identidad, identificación tributaria o pasaporte.

## Criterios de aceptación

```gherkin
Escenario: Una coincidencia nace pendiente de revisión
  Dado un screening que devuelve un nombre parecido al de la contraparte
  Cuando se registra el resultado
  Entonces la coincidencia queda en estado pendiente de revisión
  Y no se le atribuye ninguna consecuencia hasta que una persona la revise
```

```gherkin
Escenario: Descartar una coincidencia comparando identificadores
  Dado una coincidencia pendiente de revisión sobre un homónimo
  Cuando el Analista de Cumplimiento compara los identificadores y comprueba que son personas distintas
  Y la descarta indicando el fundamento
  Entonces la coincidencia queda en estado descartada
  Y quedan registrados quién la descartó, cuándo y con qué fundamento
  Y la coincidencia no desaparece del expediente
```

```gherkin
Escenario: Confirmar una coincidencia
  Dado una coincidencia pendiente de revisión cuyos identificadores corresponden a la contraparte
  Cuando el Analista de Cumplimiento la confirma indicando el fundamento
  Entonces la coincidencia queda en estado confirmada
  Y se genera la alerta correspondiente sobre el expediente
  Y la confirmación queda registrada en la bitácora
```

```gherkin
Escenario: El sistema nunca confirma por su cuenta
  Dado una coincidencia con similitud muy alta, incluso exacta en el nombre
  Cuando no ha intervenido ninguna persona
  Entonces la coincidencia sigue pendiente de revisión
  Y no existe ninguna operación automática que la confirme
```

```gherkin
Escenario: Coincidencia por variantes del nombre
  Dado una lista que contiene un nombre invertido, con un alias o con un error de digitación respecto al de la contraparte
  Cuando se ejecuta la comparación
  Entonces el parecido se detecta y se registra como coincidencia posible
  Y se indica por qué tipo de similitud se detectó
```

```gherkin
Escenario: El umbral de similitud es configuración
  Dado dos organizaciones clientes con umbrales de similitud distintos configurados
  Cuando se compara el mismo nombre en cada una
  Entonces cada una obtiene el conjunto de coincidencias que su umbral define
  Y la diferencia proviene de la configuración, no del programa
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las coincidencias
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando intenta revisar una coincidencia de "Beta Ficticia S.A.S."
  Entonces la operación es rechazada por la política de la base de datos
```

## Reglas de negocio

- Los estados de una coincidencia son cinco: `sin_coincidencia`, `posible`, `descartada`,
  `confirmada`, `pendiente_revision` (§12.4). No hay más y no se colapsan.
- **Toda coincidencia nace pendiente de revisión.** El sistema no confirma ni descarta, aunque la
  similitud sea del cien por ciento.
- Descartar o confirmar exige **fundamento registrado** y queda atribuido a una persona.
- Una coincidencia descartada **no se borra**: sigue siendo evidencia de que se revisó.
- La comparación soporta, como mínimo: exacta, por similitud, alias, nombres invertidos, errores
  de digitación, homónimos, transliteraciones e identificadores (§12.3).
- El umbral de similitud es **configuración** de la organización cliente, no una constante.
- Solo una coincidencia **confirmada** produce consecuencias sobre el expediente, y aun así a
  través de una alerta y un caso, nunca de forma directa.

## Fuera de alcance

- La ejecución del screening → `HU-025`.
- La alerta y el caso que nacen de una coincidencia confirmada → `HU-027`, `HU-028`.
- El efecto de una coincidencia confirmada sobre el nivel de riesgo → Fase 4.
- La comparación de personas relacionadas → Fase 4.
- El aprendizaje automático a partir de descartes anteriores → fase posterior.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `coincidencia.organization_id` | Sí | Organización cliente existente | No |
| `coincidencia.screening_id` | Sí | Screening que la produjo (`HU-025`) | No |
| `coincidencia.sujeto_id` | Sí | Sobre quién es la coincidencia | Sí |
| `coincidencia.registro_lista` | Sí | Entrada de la lista con la que coincidió, congelada | Sí |
| `coincidencia.tipo_similitud` | Sí | `exacta` \| `similitud` \| `alias` \| `invertido` \| `digitacion` \| `transliteracion` \| `identificador` | No |
| `coincidencia.puntaje` | Sí | Grado de similitud reportado | No |
| `coincidencia.estado` | Sí | `sin_coincidencia` \| `posible` \| `descartada` \| `confirmada` \| `pendiente_revision` | No |
| `revision_coincidencia.revisada_por` | Condicional | Obligatorio al descartar o confirmar | No |
| `revision_coincidencia.fundamento` | Condicional | Obligatorio al descartar o confirmar; texto no vacío | No |
| `revision_coincidencia.identificadores_comparados` | Sí | Qué identificadores se contrastaron | Sí |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: §12.3, §12.4, §34, §44 preguntas G y H
- Decisiones: `ADR-0005` (coincidencia, alerta y caso son tres entidades distintas)
- Modelo: `05-datos/modelo-conceptual.md`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-033` — bloqueante: qué umbral de similitud dispara una coincidencia
  y quién lo configura. Un umbral bajo inunda al analista de falsos positivos hasta que deja de
  mirarlos; uno alto deja pasar coincidencias reales. **No pasa de `borrador` hasta que se
  responda.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-025` (hay coincidencias que revisar), `HU-003` (permiso de revisar),
  `HU-006`.
- **Habilita a:** `HU-027`, `HU-028`, y el factor de riesgo por listas de la Fase 4.
- **Riesgo:** colapsar coincidencia, alerta y caso en una sola cosa es el atajo que lleva
  directamente al "rechazo automático por aparecer en una lista" que el cliente prohíbe. Son tres
  entidades y conviene que sigan siéndolo aunque al principio parezca burocracia.
