---
id: EP-006
titulo: El expediente vive
estado: borrador
capacidad: CAP-06
actualizado: 2026-08-27
---

# EP-006 — El expediente vive

## Objetivo

Que un expediente aprobado no se convierta en un archivo muerto: que la plataforma vigile lo que
cambia después de la vinculación, avise cuando algo vence o hay que actualizar, y le dé al Oficial
de Cumplimiento una vista de lo que exige su atención.

## Por qué existe

Es la Fase 6 del `02-producto/roadmap.md` y cierra las dos últimas preguntas de la §44: **O**
cuándo debe actualizarse el expediente y **P** qué ocurrió durante el monitoreo. Al terminarla
están contestadas las dieciséis, que es el criterio de producto terminado del propio cliente.

La Fase 20 marca el alcance con una frase que evita el error habitual: el monitoreo **no se limita
a volver a consultar listas**. Cubre cambios societarios, de beneficiario final, de representante
legal, de jurisdicción, documentos vencidos, nuevas sanciones y cambios en el nivel de riesgo.

Y la Fase 21 marca el otro: la periodicidad **no es una regla universal**. Sale de la metodología
del cliente, el estándar aplicable, el nivel de riesgo y los eventos.

Todo evento sigue el mismo patrón que ya existe desde `EP-003`:
`evento → alerta → caso → revisión humana → decisión`.

## Alcance

**Incluye:**

- Monitoreo continuo por evento y por periodicidad (`HU-040`).
- Vencimientos de documentos y avisos anticipados (`HU-041`).
- Renovación y actualización periódica del expediente (`HU-042`).
- Panel del Oficial de Cumplimiento (`HU-043`).
- Exportación del expediente y de reportes (`HU-044`).
- Recordatorios a la contraparte antes de que expire su enlace (`HU-045`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Monitoreo transaccional y análisis de comportamiento | Fase posterior (§36) | Fuera del MVP por decisión del cliente |
| Puntaje dinámico por comportamiento | Fase posterior (§36) | — |
| Portal para auditores externos | Fase posterior (§36) | — |
| Comparativas sectoriales | Fase posterior (§36) | — |
| Reporte a autoridad | Sin definir | No hay definición del cliente y no se inventa |
| Cobro del consumo que genera el monitoreo | Fase C | Aquí se genera el costo; cobrarlo es otra épica |

## Actores involucrados

- **Sistema** — vigila, detecta y avisa. Como siempre, no decide.
- **Analista de Cumplimiento** — atiende los casos que abre el monitoreo.
- **Oficial de Cumplimiento** — decide sobre lo que el monitoreo revela y vive en el panel.
- **Contraparte** — recibe recordatorios y aporta lo que haga falta para renovar.
- **Auditor / Consulta** — exporta y revisa.

## Criterios de éxito

1. **Ningún cambio relevante pasa desapercibido**: los eventos de la Fase 20 generan alerta.
2. **Ninguna decisión automática por monitoreo**: un evento abre un caso, nunca cierra una
   vinculación.
3. **La periodicidad se calcula**, no se fija igual para todos.
4. Al cerrar la épica, las **dieciséis preguntas** de la §44 están contestadas para cualquier
   expediente.
5. El costo del monitoreo es **previsible y visible** antes de ejecutarse.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-040` | Monitoreo continuo | Must | borrador |
| `HU-041` | Vencimientos y avisos anticipados | Must | borrador |
| `HU-042` | Renovación y actualización periódica | Must | borrador |
| `HU-043` | Panel del Oficial de Cumplimiento | Must | borrador |
| `HU-044` | Exportación del expediente y de reportes | Should | borrador |
| `HU-045` | Recordatorios a la contraparte | Could | borrador |

Orden sugerido: `HU-041` → `HU-040` → `HU-042` → `HU-043` → `HU-044` → `HU-045`. Los vencimientos
van primero porque son el trabajo programado más simple y sirven de prueba de toda la
infraestructura de tareas.

## Dependencias

- **Épicas:** `EP-000` a `EP-004`. `EP-005` no es requisito, aunque hace más útil la
  configuración de periodicidad.
- **Preguntas abiertas:** `PA-035` (periodicidad y disparadores del re-screening: es el número
  que decide el costo), `PA-039` (qué indicadores quiere el Oficial en su panel), `PA-009`
  (retención), `PA-014` (cuántas contrapartes por cliente), `PA-031` (notificaciones).
- **Decisiones:** `ADR-0001` y `08-desarrollo/arquitectura-de-aplicacion.md` (trabajos
  programados dentro de Postgres), `ADR-0005` (un cambio detectado es una afirmación nueva que
  contradice a otra).

## Riesgo abierto

**Esta es la épica que puede arruinar el margen.** El monitoreo continuo multiplica las consultas
externas por el número de contrapartes vinculadas y por la frecuencia de revisión, y la §31
impide compartir esas consultas entre clientes del SaaS. Con `PA-035` y `PA-014` abiertas, el
costo mensual recurrente por cliente no se puede estimar, y es exactamente el número que decide si
los planes de `EP-007` tienen margen o lo pierden.
