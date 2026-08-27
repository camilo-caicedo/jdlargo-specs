---
id: HU-025
titulo: Screening contra listas, PEP y sanciones
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-025 — Screening contra listas, PEP y sanciones

## Historia

**Como** Analista de Cumplimiento
**quiero** que el sistema consulte a la contraparte contra las listas que mi organización tiene
configuradas y guarde la respuesta completa
**para** poder demostrar qué se consultó, cuándo y con qué resultado, aunque la lista haya
cambiado desde entonces.

## Contexto

Es la Fase 12 del documento del cliente, y su §12.1 corrige un error de la versión anterior:
**no limitarse a las dos listas internacionales más conocidas**. El alcance configurable incluye
listas vinculantes en Colombia, listas de organismos internacionales, sanciones administrativas,
antecedentes, fuentes reputacionales, listas internas del propio cliente, listas de terceros
autorizados y fuentes comerciales.

La §12.2 añade la identificación de **PEP**: si la persona ocupa u ocupó un cargo público
relevante.

Lo que esta historia produce son **coincidencias**, y nada más. Convertirlas en algo requiere una
persona: es `HU-026` y `HU-028`. La §34 lo prohíbe expresamente: nada de declarar positivos
automáticos ni de rechazar por aparecer en una lista.

## Criterios de aceptación

```gherkin
Escenario: Ejecutar un screening y congelar su resultado
  Dado un expediente con la contraparte identificada
  Y un conjunto de listas configuradas en el catálogo de fuentes
  Cuando se ejecuta el screening
  Entonces queda registrado como evento inmutable con la respuesta completa tal como llegó
  Y con la versión del conjunto de datos consultado, la fecha, la hora y el costo
  Y con qué listas se consultaron exactamente
```

```gherkin
Escenario: El screening produce coincidencias, no conclusiones
  Dado un screening ejecutado que devuelve nombres parecidos
  Cuando se registra el resultado
  Entonces cada parecido queda como una coincidencia en estado pendiente de revisión
  Y ninguna queda marcada como confirmada
  Y el expediente no cambia de decisión por ese hecho
```

```gherkin
Escenario: Sin coincidencias también es un resultado que se guarda
  Dado un screening que no devuelve ningún parecido
  Cuando se registra el resultado
  Entonces queda constancia de que se consultó y de que no hubo coincidencias
  Y esa constancia es evidencia igual que cualquier otra
```

```gherkin
Escenario: Identificación de PEP
  Dado una fuente configurada que reporta condición de Persona Expuesta Políticamente
  Cuando el screening devuelve esa condición para la contraparte
  Entonces se registra como coincidencia del tipo PEP, pendiente de revisión
  Y no se asume por sí sola ninguna consecuencia sobre el expediente
```

```gherkin
Escenario: Nunca hay rechazo automático por aparecer en una lista
  Dado una coincidencia de cualquier tipo devuelta por el screening
  Cuando se registra
  Entonces no existe ningún camino por el que el expediente quede rechazado sin decisión humana
  Y la coincidencia genera una alerta, que se gestiona como caso
```

```gherkin
Escenario: El resultado sobrevive al cambio de la lista
  Dado un screening ejecutado hace meses
  Cuando la lista consultada cambia su contenido
  Entonces la evidencia guardada sigue mostrando lo que la lista decía en el momento de la consulta
  Y la evidencia anterior no se actualiza ni se reemplaza
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el screening
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta screenings con su contexto de usuario propagado
  Entonces obtiene únicamente los de expedientes de "Alfa Ficticia S.A.S."
  Y no obtiene ninguno de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Qué listas se consultan es **configuración** de cada organización cliente (`HU-023`), y no se
  limita a las internacionales más conocidas (§12.1).
- Cada screening se persiste como **evento inmutable** con la respuesta completa —no un enlace—,
  la versión del conjunto de datos, el modelo o proveedor usado, el costo de la llamada y quién
  lo disparó (`ADR-0001` §18).
- El screening produce **coincidencias en estado pendiente de revisión**. No confirma, no
  descarta y no decide.
- **Ningún rechazo automático por aparecer en una lista** (§34).
- Un resultado sin coincidencias también se guarda: demostrar que se consultó es parte del
  cumplimiento verificable (§42).
- La consulta ocurre en el servidor, con clave de idempotencia, y **la cuota se verifica antes de
  gastar** cuando exista la capa comercial (Fase C).
- Las listas internas del propio cliente son una fuente más del catálogo, con las mismas reglas.

## Fuera de alcance

- La comparación de nombres y sus estados → `HU-026`.
- La gestión de la alerta resultante → `HU-027` y `HU-028`.
- El re-screening periódico y el disparado por evento → Fase 6.
- El screening de personas relacionadas y beneficiarios finales → Fase 4.
- El control de cupo y su bloqueo → Fase C.
- El análisis avanzado de medios adversos → fase posterior (§36).

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `screening.organization_id` | Sí | Organización cliente existente | No |
| `screening.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `screening.sujeto_id` | Sí | Sobre quién se consultó | Sí |
| `screening.fuentes` | Sí | Al menos una fuente del catálogo | No |
| `screening.ejecutado_en` | Sí | Momento; se escribe una sola vez | No |
| `screening.disparado_por` | Sí | Usuario o proceso automático | No |
| `screening.respuesta` | Sí | Respuesta completa congelada | Sí |
| `screening.version_datos` | No | Versión del conjunto de datos, si la fuente la reporta | No |
| `screening.costo` | Sí | Costo de la consulta | No |
| `screening.clave_idempotencia` | Sí | Impide consultar y cobrar dos veces un reintento | No |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: Fase 12 (§12.1 y §12.2), §34, §42, §44 preguntas F y G
- Decisiones: `ADR-0001` (evento inmutable con snapshot, versión de dataset y costo),
  `ADR-0005` (la coincidencia no es una alerta y la alerta no es un caso)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-005` (qué fuentes de listas), `PA-012` (costo por consulta del
  proveedor local), `PA-014` (cuántas contrapartes maneja un cliente típico). **Queda en
  `borrador`.**
- **Supuestos:** `SUP-003`.
- **Depende de:** `HU-023` (catálogo), `HU-008` (hay un sujeto), `HU-026` (los resultados se
  comparan), `HU-027` (las coincidencias generan alertas).
- **Habilita a:** el motor de riesgo de la Fase 4 y el monitoreo de la Fase 6.
- **Riesgo:** es la operación más cara del producto y la que más se repite. Sin `PA-005` y
  `PA-012` no se puede estimar el costo por expediente, que es el número del que depende todo el
  modelo comercial.
