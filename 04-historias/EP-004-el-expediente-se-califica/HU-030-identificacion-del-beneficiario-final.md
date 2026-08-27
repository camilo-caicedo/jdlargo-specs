---
id: HU-030
titulo: Identificación del beneficiario final
estado: borrador
epica: EP-004
prioridad: Must
actualizado: 2026-08-27
---

# HU-030 — Identificación del beneficiario final

## Historia

**Como** Oficial de Cumplimiento
**quiero** que el sistema me proponga quién parece ser el beneficiario final y me muestre en qué
se basa
**para** decidirlo yo, sabiendo qué parte está respaldada por documentos y qué parte sigue siendo
una suposición.

## Contexto

El glosario del proyecto ya fija la distinción y la Fase 10 la desarrolla: el beneficiario final
es quien realmente controla o se beneficia de una empresa, **no es lo mismo que accionista**, y
su determinación **no se da por completa solo con el certificado de existencia y representación
legal**. Ese documento ayuda a identificar administradores y parte de la estructura societaria,
pero puede no bastar.

El documento del cliente permite que el sistema sugiera —"falta verificar a Persona 3 como
beneficiario final"— y es tajante en que **esa determinación siempre queda sujeta a revisión
humana**. La §34 lo repite entre lo que el MVP no debe hacer: determinar beneficiarios finales
sin revisión humana.

Y la §23 pide atención especial en la auditoría a los **cambios de beneficiario final**: es uno
de los hechos que más importa poder reconstruir.

## Criterios de aceptación

```gherkin
Escenario: El sistema sugiere un beneficiario final
  Dado un grafo de relaciones con participaciones societarias registradas
  Cuando el sistema evalúa quién podría ser beneficiario final según el criterio configurado
  Entonces registra una sugerencia con las relaciones y los porcentajes en los que se basa
  Y la sugerencia queda como propuesta, no como determinación
```

```gherkin
Escenario: Una sugerencia no equivale a una determinación
  Dado una sugerencia de beneficiario final registrada
  Cuando no ha intervenido ninguna persona
  Entonces el expediente muestra que el beneficiario final está pendiente de determinar
  Y ninguna evaluación de riesgo lo trata como determinado
```

```gherkin
Escenario: El Oficial de Cumplimiento determina el beneficiario final
  Dado una sugerencia y su evidencia
  Cuando el Oficial de Cumplimiento determina quién es el beneficiario final indicando el fundamento
  Entonces queda registrada la determinación con su nombre, su cargo, la fecha y el fundamento
  Y queda registrado en qué evidencia se apoyó
  Y el hecho aparece en la bitácora
```

```gherkin
Escenario: Un solo documento societario no basta
  Dado una estructura societaria conocida únicamente por el certificado de existencia y representación legal
  Cuando se evalúa si el beneficiario final está determinado
  Entonces el sistema indica que esa evidencia no es suficiente por sí sola
  Y señala qué información falta
```

```gherkin
Escenario: Estructura que no permite llegar al beneficiario final
  Dado una cadena societaria que no se puede recorrer hasta una persona natural con la información disponible
  Cuando se evalúa el expediente
  Entonces el beneficiario final queda como no determinable con la evidencia actual
  Y se genera una alerta
  Y esa condición queda disponible como factor para la metodología de riesgo
```

```gherkin
Escenario: Cambio de beneficiario final
  Dado un expediente con beneficiario final ya determinado
  Cuando se determina uno distinto con fundamento
  Entonces la determinación anterior se conserva completa
  Y el cambio queda registrado en la bitácora con atención especial
  Y se genera una alerta sobre el expediente
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el beneficiario final
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta determinaciones de beneficiario final con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- El beneficiario final **se determina, no se calcula**. El sistema sugiere; una persona
  autorizada determina, con fundamento y evidencia registrados.
- Una determinación es un **evento inmutable** (`ADR-0005`), con persona, cargo, fecha,
  fundamento y evidencia. Cambiarla es determinar de nuevo, no editar.
- **El certificado societario no basta por sí solo.** El criterio de suficiencia es configuración
  del cliente, y la plataforma señala cuándo no se cumple.
- Cuando la estructura no permite llegar a una persona natural, el beneficiario final queda como
  **no determinable con la evidencia actual**, se genera alerta, y esa condición está disponible
  como factor de riesgo.
- Los cambios de beneficiario final se auditan con atención especial (§23) y generan alerta.
- El umbral de participación que sugiere un beneficiario final es **configuración**, no una
  constante del programa.

## Fuera de alcance

- El registro de las relaciones que alimentan el análisis → `HU-029`.
- La consulta a registros oficiales de beneficiarios finales, mientras `PA-005` no defina si
  existe una fuente disponible y a qué costo.
- El efecto del beneficiario final sobre el nivel de riesgo → `HU-032`.
- La diligencia sobre el beneficiario final como sujeto: la determina el motor de relaciones
  (`HU-029`) y se ejecuta con `EP-001`.
- El seguimiento de cambios societarios después de la vinculación → Fase 6.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `beneficiario_final.organization_id` | Sí | Organización cliente existente | No |
| `beneficiario_final.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `beneficiario_final.sujeto_id` | Condicional | Obligatorio si se determinó a alguien | Sí |
| `beneficiario_final.estado` | Sí | `sugerido` \| `determinado` \| `no_determinable` | No |
| `beneficiario_final.fundamento` | Condicional | Obligatorio al determinar; texto no vacío | No |
| `beneficiario_final.evidencia` | Condicional | Obligatoria al determinar; relaciones y documentos citados | Sí |
| `beneficiario_final.determinado_por` y `cargo` | Condicional | Obligatorios al determinar | No |
| `beneficiario_final.determinado_en` | Condicional | Momento; se escribe una sola vez | No |
| `criterio_suficiencia` | Sí | Configuración: qué evidencia se considera suficiente | No |
| `umbral_participacion` | Sí | Configuración: desde qué porcentaje se sugiere | No |

## Trazabilidad

- Épica: `EP-004`
- Capacidad: `CAP-04`
- Documento del cliente: Fase 10, §23, §34, §37
- Decisiones: `ADR-0005` (la determinación es un evento inmutable), `ADR-0004` (los criterios son
  configuración)
- Glosario: **Beneficiario final**

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-005` — si existe una fuente consultable de beneficiarios finales y
  a qué costo. `PA-018` — qué criterios de suficiencia usa el primer cliente. **Queda en
  `borrador`.**
- **Supuestos:** `SUP-006`.
- **Depende de:** `HU-029` (el grafo), `HU-024` (evidencia verificada), `HU-027` (alertas).
- **Habilita a:** `HU-032` (factor de riesgo por estructura y por beneficiario final poco claro),
  `HU-033` (es una de las causales de diligencia intensificada).
- **Riesgo:** es el punto donde una sugerencia bien presentada se convierte, en la práctica, en la
  determinación. Si la interfaz muestra el nombre sugerido como si ya estuviera decidido, la
  revisión humana existe en el modelo de datos y no en la realidad.
