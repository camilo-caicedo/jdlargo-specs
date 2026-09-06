---
id: HU-054
titulo: Autenticar la integración y dejar bitácora de cada extracción
estado: borrador
epica: EP-008
prioridad: Must
actualizado: 2026-09-06
---

# HU-054 — Autenticar la integración y dejar bitácora de cada extracción

## Historia

**Como** Administrador de una organización cliente
**quiero** generar y revocar credenciales para la integración con mi otro sistema, y ver un
registro de cada vez que se sacó información de mis expedientes
**para** saber quién o qué sistema extrajo información, y poder cortar el acceso si hace falta.

## Contexto

Tanto la exportación por archivo (`HU-052`) como la interfaz de programación (`HU-053`) necesitan
lo mismo por debajo: algo que autentique quién pide la extracción, y un registro de que ocurrió.
Esta historia construye esa base común, con el mismo patrón que ya usa el catálogo de fuentes
externas (`HU-023`): credenciales por organización cliente, nunca compartidas, y todo evento —
exitoso o rechazado— en la bitácora transversal (`HU-006`).

## Criterios de aceptación

```gherkin
Escenario: Generar una credencial de integración
  Dado un Administrador de una organización cliente
  Cuando genera una credencial de integración
  Entonces el sistema la muestra completa una única vez
  Y a partir de ahí solo se puede identificar por su referencia, nunca reconstruir su valor
```

```gherkin
Escenario: Revocar una credencial
  Dado una credencial de integración vigente
  Cuando el Administrador la revoca
  Entonces queda inutilizable de inmediato
  Y la revocación queda registrada en la bitácora con quién la revocó y cuándo
```

```gherkin
Escenario: Una extracción con credencial revocada se rechaza
  Dado una credencial ya revocada
  Cuando se intenta usarla para exportar o consultar un expediente
  Entonces la operación se rechaza
  Y el intento queda registrado en la bitácora como rechazado
```

```gherkin
Escenario: Toda extracción exitosa queda en bitácora
  Dado una extracción exitosa, por archivo o por interfaz de programación
  Cuando se completa
  Entonces la bitácora registra la organización, el expediente, los campos entregados, el medio
  usado, quién o qué credencial la pidió, y el momento
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las credenciales
  Dado una credencial de integración de "Alfa Ficticia S.A.S."
  Cuando se usa para intentar cualquier operación sobre datos de "Beta Ficticia S.A.S."
  Entonces la operación se rechaza
  Y queda registrada como un intento rechazado
```

## Reglas de negocio

- Las credenciales de integración son **por organización cliente**, nunca compartidas entre
  organizaciones (mismo patrón que `HU-023` con las credenciales de fuentes externas).
- Una credencial se puede revocar en cualquier momento; la revocación es inmediata y no requiere
  esperar a que expire.
- El valor completo de una credencial se muestra **una sola vez**, al crearla. A partir de ahí el
  sistema solo guarda su huella, nunca el valor en claro.
- Toda extracción —por archivo (`HU-052`) o por interfaz de programación (`HU-053`)— deja un
  registro en la misma bitácora transversal que usa el resto de la plataforma (`HU-006`),
  incluidos los intentos rechazados.

## Fuera de alcance

- Rotación automática programada de credenciales.
- Límites de tasa (*rate limiting*) detallados: `TBD`.
- Un panel dedicado de monitoreo de integraciones: por ahora la bitácora general y el panel del
  Oficial de Cumplimiento (`HU-043`) son donde se consulta esta información; una vista propia
  queda para una fase posterior si el volumen lo justifica.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `credencial_integracion.organization_id` | Sí | Organización cliente existente | No |
| `credencial_integracion.hash` | Sí | Nunca se guarda en claro | Sí |
| `credencial_integracion.creada_en` / `creada_por` | Sí | Usuario que la generó | No |
| `credencial_integracion.revocada_en` / `revocada_por` | No | Nula mientras esté vigente | No |
| `bitacora_extraccion.organization_id` | Sí | Organización cliente existente | No |
| `bitacora_extraccion.expediente_id` | Sí | Expediente al que se accedió | No |
| `bitacora_extraccion.medio` | Sí | `archivo` \| `interfaz_programacion` | No |
| `bitacora_extraccion.resultado` | Sí | `entregado` \| `rechazado`, con motivo | No |
| `bitacora_extraccion.actor` | Sí | Usuario o credencial de integración que la generó | No |

## Trazabilidad

- Épica: `EP-008`
- Capacidad: `CAP-08`
- Decisiones: `ADR-0007` (bitácora append-only con hash por evento)

## Dependencias y riesgos

- **Preguntas abiertas:** **`PA-046`** — sin saber qué medio(s) se construyen (`HU-052`,
  `HU-053`, o ambos), esta historia no puede fijar del todo qué tipos de evento registrar.
- **Depende de:** `HU-006` (bitácora inmutable transversal), `HU-051` (campos exportables).
- **Habilita a:** `HU-052`, `HU-053`.
- **Riesgo:** ninguno propio más allá del de toda la épica (`PA-046`); es la pieza menos afectada
  por cuál medio se elija, porque autenticación y bitácora hacen falta en cualquiera de los dos.
