---
id: HU-046
titulo: Planes, licencias y cupo de consultas
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-09-06
---

# HU-046 — Planes, licencias y cupo de consultas

> **Actualización 2026-09-05 (`PA-015`, `PA-036`, `PA-038`).** La "licencia" **no es la métrica
> única**. El modelo es híbrido: `suscripción base + usuarios incluidos + cupo de consultas o
> créditos + módulos premium + excedente por consumo`. Tres planes de referencia —Starter,
> Professional, Enterprise— con límites por usuarios, contrapartes, consultas, OCR/IA y monitoreo.
> **Los planes son configuración, no código.** Y el alta de organizaciones es **manual al
> principio**, híbrida después: **el MVP no necesita registro público**.

## Historia

**Como** Cliente del SaaS
**quiero** contratar un plan que incluya un número de consultas al mes y saber en qué consiste
**para** poder prever lo que voy a pagar en vez de recibir una factura sorpresa.

## Contexto

`PA-015` está resuelta: **la licencia incluye un cupo de consultas mensuales**; si el cliente lo
supera, sube de plan o paga por consulta individual. El objetivo declarado es controlar precio,
gasto y margen.

Esta historia construye el modelo de planes y la asignación a cada organización cliente. Lo que
todavía no existe son los planes concretos: cuántos, a qué precio y con qué cupo es `PA-036`, sin
responder. Aquí se construye el envase, con el contenido marcado como pendiente.

Y hay una restricción que viene de `EP-000` y aplica igual aquí: el plan de una organización
cliente es un dato suyo, con su aislamiento como cualquier otro.

## Criterios de aceptación

```gherkin
Escenario: Asignar un plan a una organización cliente
  Dado un plan definido con su cupo de consultas y su precio
  Cuando se le asigna a una organización cliente
  Entonces la organización queda con ese plan vigente desde una fecha
  Y su cupo mensual queda disponible para el ciclo en curso
  Y la asignación queda registrada en la bitácora
```

```gherkin
Escenario: Consultar el consumo frente al cupo
  Dado una organización cliente con plan vigente y consumo del ciclo
  Cuando el Administrador consulta su estado
  Entonces ve cuántas consultas lleva, cuántas incluye su plan y cuántas le quedan
  Y ve desde y hasta cuándo va el ciclo en curso
```

```gherkin
Escenario: Cambiar de plan
  Dado una organización cliente con un plan vigente
  Cuando se le asigna un plan distinto
  Entonces el plan anterior se conserva en el historial con su periodo de vigencia
  Y el cupo aplicable a cada consumo es el del plan vigente en el momento de consumir
```

```gherkin
Escenario: El cupo se renueva con el ciclo
  Dado una organización cliente que consumió parte de su cupo
  Cuando cierra el ciclo y empieza el siguiente
  Entonces el cupo del ciclo nuevo está completo
  Y el consumo del ciclo anterior queda registrado y cerrado
```

```gherkin
Escenario: Un plan no se modifica en sitio
  Dado un plan con clientes asignados
  Cuando se necesita cambiar su precio o su cupo
  Entonces se define una versión nueva del plan
  Y los ciclos ya cerrados conservan las condiciones con las que se calcularon
```

```gherkin
Escenario: La organización individual es un cliente más
  Dado una persona que contrata por su cuenta
  Cuando se le asigna un plan
  Entonces se le trata como una organización cliente de un miembro
  Y no existe ningún caso especial en el modelo
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre planes y consumo
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta su plan y su consumo con su contexto de usuario propagado
  Entonces obtiene únicamente los suyos
  Y no obtiene ningún dato de plan ni de consumo de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Un plan define: cupo de consultas por ciclo, precio del ciclo, precio de la consulta excedente y
  condiciones. **Los valores concretos son `TBD` hasta `PA-036`.**
- Una organización cliente tiene **un plan vigente a la vez**, con su periodo de vigencia. El
  historial de planes se conserva.
- El cupo aplicable a un consumo es el del plan vigente **en el momento de consumir**, no el
  actual.
- Los planes no se editan cuando ya tienen ciclos calculados: se define una versión nueva. Es la
  misma regla de inmutabilidad de `ADR-0004`, aplicada a lo comercial.
- Toda organización cliente tiene un plan vigente, incluida una organización de un solo
  miembro (persona individual con su propia membresía): no hay un modelo de facturación aparte
  para el caso individual (`SUP-005` **derribado y reemplazado** — modelo confirmado
  `Usuario › Membresía › Organización` en `actores-y-roles.md`).
- El plan y el consumo son datos de la organización cliente, con su aislamiento (`HU-002`).

## Fuera de alcance

- La medición del consumo en sí → `HU-047`.
- El cierre de ciclo y el excedente → `HU-048`.
- El cobro → `HU-049`.
- El registro público de clientes nuevos, mientras `PA-038` no defina si el alta es por
  autoservicio o manual.
- Periodos de prueba, prorrateo y descuentos.
- Planes con cupos por tipo de fuente: el modelo admite un cupo global mientras el cliente no pida
  más granularidad.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `plan.name` | Sí | Único | No |
| `plan.query_quota` | Sí | Entero positivo `(TBD — PA-036)` | No |
| `plan.billing_cycle_price` | Sí | Monto y moneda `(TBD — PA-036)` | No |
| `plan.overage_price` | Sí | Monto por consulta excedente `(TBD — PA-036)` | No |
| `plan.version` | Sí | Correlativa; no se edita una versión con ciclos calculados | No |
| `subscription.organization_id` | Sí | Organización cliente existente | No |
| `subscription.plan_version_id` | Sí | Versión de plan existente | No |
| `subscription.active_from` / `active_to` | Sí | Una sola suscripción vigente a la vez | No |
| `billing_cycle.start_date` / `end_date` | Sí | Periodo mensual por organización cliente | No |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` (modelo de cupo más excedente), `ADR-0001` (stack técnico y aislamiento por organización)
- Preguntas: cierra la implementación de `PA-015`

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-043`** — precios y cupos concretos de cada plan, que dependen de
  `PA-040`. `PA-015`, `PA-036` y `PA-038` **resueltas** → `ADR-0002` §2b y §2c.
- **Depende de:** `HU-001`, `HU-002`.
- **Habilita a:** `HU-047` a `HU-050`.
- **Riesgo:** fijar el cupo de un plan sin saber cuánto cuesta una consulta (`PA-012`), cuántas
  contrapartes tiene un cliente (`PA-014`) y con qué frecuencia se re-consultan (`PA-035`) es
  fijar el margen a ciegas. Los tres números hay que tenerlos antes de publicar precios, no
  después.
