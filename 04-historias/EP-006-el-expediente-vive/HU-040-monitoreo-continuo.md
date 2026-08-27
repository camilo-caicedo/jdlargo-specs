---
id: HU-040
titulo: Monitoreo continuo
estado: borrador
epica: EP-006
prioridad: Must
actualizado: 2026-08-27
---

# HU-040 — Monitoreo continuo

## Historia

**Como** Oficial de Cumplimiento
**quiero** enterarme cuando algo cambia en una contraparte que ya vinculamos
**para** no descubrir dentro de un año que un proveedor entró en una lista o cambió de dueño el
mes siguiente a haberlo aprobado.

## Contexto

Es la Fase 20, y su primera advertencia es la que evita construir la versión pobre: el monitoreo
**no se limita a volver a consultar listas**. Los eventos que debe cubrir son nueva coincidencia
en listas, cambio en la condición de PEP, cambio societario, cambio de beneficiario final, cambio
de representante legal, cambio de jurisdicción, documento vencido, cambio de cuenta bancaria,
nueva sanción, nueva alerta, o cambio en el nivel de riesgo calculado.

Cada evento sigue el patrón que ya existe: `evento → alerta → caso → revisión humana → decisión`.

`ADR-0005` ya dejó preparada la mecánica: un cambio detectado **es una afirmación nueva que
contradice a una anterior**. No hace falta un modelo aparte para el monitoreo; es el mismo, en el
tiempo.

Y el §46 añade un caso que no admite espera: una coincidencia en listas después de vincular
**dispara alerta inmediata**, no espera al ciclo de actualización.

## Criterios de aceptación

```gherkin
Escenario: Re-consulta periódica según la metodología
  Dado un expediente vinculado con su periodicidad calculada
  Cuando llega el momento de la revisión programada
  Entonces se ejecutan las consultas que la configuración defina para ese expediente
  Y cada consulta queda registrada con su evidencia y su costo
```

```gherkin
Escenario: Una coincidencia nueva dispara alerta inmediata
  Dado una contraparte ya vinculada
  Cuando una consulta de monitoreo devuelve una coincidencia que antes no existía
  Entonces se genera una alerta de inmediato, sin esperar al ciclo de actualización
  Y la coincidencia entra en el mismo flujo de revisión humana de la Fase 3
```

```gherkin
Escenario: Un cambio detectado es una afirmación nueva
  Dado un expediente cuyo representante legal estaba registrado
  Cuando una consulta de monitoreo devuelve otro representante legal
  Entonces se registra una afirmación nueva con origen verificado y su momento
  Y la anterior se conserva
  Y se abre una discrepancia y su alerta
```

```gherkin
Escenario: El monitoreo no decide
  Dado un evento de monitoreo de cualquier gravedad
  Cuando se registra
  Entonces el expediente no cambia su decisión de vinculación
  Y se abre un caso para que una persona lo analice
```

```gherkin
Escenario: Qué se monitorea es configuración
  Dado dos organizaciones clientes con eventos y periodicidades distintas configuradas
  Cuando se ejecuta el monitoreo en cada una
  Entonces cada una vigila lo que su configuración define
  Y la diferencia no proviene del programa
```

```gherkin
Escenario: El monitoreo no se ejecuta sobre expedientes que no lo requieren
  Dado un expediente rechazado o cerrado sin vinculación
  Cuando corre el ciclo de monitoreo
  Entonces ese expediente no genera consultas
  Y no se incurre en costo por él
```

```gherkin
Escenario: Un fallo del proveedor no rompe el ciclo
  Dado un proveedor de fuentes indisponible durante una ejecución programada
  Cuando falla la consulta de un expediente
  Entonces el resto del ciclo continúa
  Y el expediente afectado queda marcado para reintento
  Y el fallo queda registrado
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el monitoreo
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta eventos de monitoreo con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y ninguna consulta de monitoreo de una organización cliente reutiliza resultados de otra
```

## Reglas de negocio

- El monitoreo cubre **todos los eventos de la Fase 20**, no solo la re-consulta de listas.
- Un evento produce **alerta y caso**; nunca una decisión (§34).
- Un cambio detectado se registra como **afirmación nueva** con su origen y su momento
  (`ADR-0005`). La anterior no se borra.
- Qué se monitorea, con qué periodicidad y con qué disparadores es **configuración** de la
  organización cliente (`ADR-0004`).
- Una coincidencia nueva en listas **no espera** al ciclo: alerta inmediata (§46).
- Solo se monitorean los expedientes que la configuración determine. Monitorear lo que no hace
  falta es costo puro.
- Cada consulta de monitoreo registra su costo, como cualquier otra (`HU-023`), y **no se comparte
  entre organizaciones clientes** (§31).
- Los trabajos programados corren dentro de Postgres con `pg_cron` y `pgmq`, sin infraestructura
  adicional (`ADR-0001`), y son reintentables e idempotentes.

## Fuera de alcance

- Los vencimientos de documentos, que tienen su propia mecánica → `HU-041`.
- La renovación del expediente → `HU-042`.
- El monitoreo transaccional y el análisis de comportamiento → fase posterior (§36).
- El control de cupo antes de gastar en monitoreo → Fase C.
- El reporte a autoridad derivado de un hallazgo: sin definición del cliente.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `evento_monitoreo.organization_id` | Sí | Organización cliente existente | No |
| `evento_monitoreo.expediente_id` | Sí | Expediente vinculado de esa organización | No |
| `evento_monitoreo.tipo` | Sí | `coincidencia` \| `pep` \| `societario` \| `beneficiario_final` \| `representante_legal` \| `jurisdiccion` \| `documento_vencido` \| `cuenta_bancaria` \| `sancion` \| `riesgo` | No |
| `evento_monitoreo.detectado_en` | Sí | Momento; se escribe una sola vez | No |
| `evento_monitoreo.evidencia` | Sí | Respuesta congelada de la fuente | Sí |
| `evento_monitoreo.alerta_id` | Sí | Alerta generada | No |
| `programacion.periodicidad` | Sí | Calculada según `HU-042`; nunca fija para todos | No |
| `ejecucion_monitoreo.costo` | Sí | Costo de las consultas ejecutadas | No |
| `ejecucion_monitoreo.clave_idempotencia` | Sí | Impide consultar y cobrar dos veces un reintento | No |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: Fase 20, §31, §34, §46, §44 pregunta P
- Decisiones: `ADR-0001` (trabajos programados en Postgres), `ADR-0005` (un cambio es una
  afirmación nueva), `07-integraciones/README.md` (riesgo de costo)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-035` — bloqueante: con qué periodicidad y ante qué disparadores se
  re-consulta. Es la principal fuente de costo variable del producto. `PA-014` — cuántas
  contrapartes maneja un cliente típico. `PA-005` y `PA-012` — qué fuentes y a qué precio. **No
  pasa de `borrador` hasta que se responda `PA-035`.**
- **Supuestos:** `SUP-002`.
- **Depende de:** `HU-025`, `HU-026`, `HU-027`, `HU-028`, `HU-031`, `HU-005`.
- **Habilita a:** `HU-042` (los eventos disparan actualización fuera de calendario) y `HU-043`.
- **Riesgo:** es la funcionalidad que convierte un negocio por consulta en uno por suscripción, y
  también la que puede consumir el margen entero. Conviene poder apagarla por cliente y por tipo
  de contraparte desde el primer día.
