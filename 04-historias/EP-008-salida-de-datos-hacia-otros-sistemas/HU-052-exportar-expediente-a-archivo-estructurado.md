---
id: HU-052
titulo: Exportar un expediente cerrado a archivo estructurado (JSON/TXT)
estado: borrador
epica: EP-008
prioridad: Must
actualizado: 2026-09-06
---

# HU-052 — Exportar un expediente cerrado a archivo estructurado (JSON/TXT)

## Historia

**Como** Administrador de una organización cliente
**quiero** descargar un expediente cerrado en un archivo JSON o TXT, con el formato que configuré
para mi organización
**para** cargarlo en mi otro sistema sin copiar los datos a mano.

## Contexto

`PA-010` propone el archivo estructurado como una de las dos vías posibles de salida (la otra es
la interfaz de programación, `HU-053`). Esta historia cubre el caso más simple: un Administrador
pide, bajo demanda, el archivo de un expediente ya cerrado.

Esta historia **no** es lo mismo que `HU-044` (Fase 6, exportación de reportes en PDF/Excel/CSV
para el Auditor). `HU-044` produce documentos para que una persona los lea; esta historia produce
un archivo estructurado para que **otro sistema** lo consuma.

## Criterios de aceptación

```gherkin
Escenario: Exportar un expediente cerrado
  Dado un expediente con decisión tomada y cerrado
  Cuando el Administrador solicita su exportación
  Entonces se genera un archivo en el formato configurado para su organización (JSON o TXT)
  Y el archivo contiene únicamente los campos marcados como exportables
```

```gherkin
Escenario: No se puede exportar un expediente que no está cerrado
  Dado un expediente en cualquier estado distinto de cerrado
  Cuando se solicita su exportación
  Entonces el sistema rechaza la solicitud
  Y explica que solo se exportan expedientes cerrados
```

```gherkin
Escenario: El archivo respeta la configuración de campos exportables
  Dado una organización cliente con solo algunos campos marcados como exportables
  Cuando se exporta un expediente suyo
  Entonces el archivo no contiene ningún campo que no esté marcado como exportable
```

```gherkin
Escenario: Aislamiento entre organizaciones
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando solicita la exportación de un expediente con su contexto de usuario propagado
  Entonces solo puede exportar expedientes de su propia organización
  Y un intento sobre un expediente de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Solo se exportan **expedientes cerrados** (con decisión tomada): un expediente a medio evaluar
  no sale como si fuera definitivo.
- Solo salen los campos marcados exportables por `HU-051`.
- El formato (JSON o TXT) y su esquema exacto son **configurables por organización cliente**. El
  esquema concreto de cada formato es `TBD` — depende de `PA-046` (qué sistema destino existe y
  qué formato espera).
- Cada exportación queda registrada en la bitácora (`HU-054`).

## Fuera de alcance

- El disparo automático, programado o por evento (p. ej. al cerrar el expediente) — depende de
  `PA-047`. Esta historia cubre exclusivamente el disparo **manual, bajo demanda**.
- La interfaz de programación → `HU-053`.
- Un conector a un sistema destino específico (ERP, TMS): el archivo es genérico, no está
  adaptado a ningún sistema en particular.
- Exportación masiva de varios expedientes a la vez.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `exportacion.organization_id` | Sí | Organización cliente existente | No |
| `exportacion.expediente_id` | Sí | Expediente en estado cerrado | No |
| `exportacion.formato` | Sí | `json` \| `txt`, según configuración de la organización `(TBD — PA-046)` | No |
| `exportacion.campos_incluidos` | Sí | Subconjunto de los campos marcados exportables (`HU-051`) | Depende del campo |
| `exportacion.solicitado_por` | Sí | Usuario que la pidió | No |
| `exportacion.fecha` | Sí | Momento de la generación | No |

## Trazabilidad

- Épica: `EP-008`
- Capacidad: `CAP-08`
- Preguntas: implementa una de las dos vías propuestas en `PA-010`

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-046`** — sin saber si el archivo estructurado es en efecto el
  medio que el cliente quiere (y con qué esquema), esta historia se puede escribir pero no
  construir a ciegas. **`PA-047`** — define si esta historia necesita crecer hacia disparo
  programado o por evento.
- **Depende de:** `HU-051` (campos exportables), `HU-009` (estados del expediente, para saber qué
  es "cerrado").
- **Riesgo:** construir el exportador antes de `PA-046` arriesga construir un esquema de archivo
  que el sistema destino real no pueda leer sin cambios.
