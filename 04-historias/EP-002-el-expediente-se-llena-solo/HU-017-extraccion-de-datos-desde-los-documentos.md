---
id: HU-017
titulo: Extracción de datos desde los documentos
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-08-27
---

# HU-017 — Extracción de datos desde los documentos

## Historia

**Como** Analista de Cumplimiento
**quiero** que el sistema lea los documentos que entregó la contraparte y proponga los campos
que encuentra
**para** dejar de transcribir a mano lo que ya está escrito, sin que eso signifique dar por
cierto lo que leyó una máquina.

## Contexto

Es la Fase 8 del documento del cliente. La IA puede detectar el tipo de documento, extraer
campos, identificar fechas y cifras, y señalar inconsistencias. Cada dato extraído se guarda
con su valor, el documento fuente, la ubicación dentro del documento cuando se puede
determinar, el nivel de confianza, el modelo y versión que lo produjo, y el momento.

`ADR-0001` fija la estrategia: **plantillas deterministas** para los formatos colombianos
conocidos —documento de identidad, certificado de existencia y representación legal, registro
tributario— y **modelo multimodal como respaldo** para todo lo demás. El motor vive detrás de un
puerto reemplazable, ahora por dos razones: la técnica y la de transferencia internacional de
datos (§33).

Y la regla que gobierna todo lo demás, de `ADR-0005`: lo extraído es un **origen**, no una
verdad. Nunca reemplaza lo declarado.

## Criterios de aceptación

```gherkin
Escenario: Extraer campos de un documento cargado
  Dado un documento en estado recibido en un expediente
  Cuando el sistema ejecuta la extracción sobre él
  Entonces se registran las afirmaciones encontradas con origen extraído
  Y cada una cita el documento fuente, su nivel de confianza y la ejecución de IA que la produjo
  Y ninguna afirmación declarada existente se modifica ni se borra
```

```gherkin
Escenario: Lo extraído no desplaza a lo declarado
  Dado un campo con una afirmación declarada por la contraparte
  Cuando la extracción produce un valor distinto para ese mismo campo
  Entonces existen dos afirmaciones vigentes, con su origen cada una
  Y el sistema no elige ninguna por su cuenta
  Y la diferencia queda visible como discrepancia
```

```gherkin
Escenario: Extracción con confianza insuficiente
  Dado un campo extraído con un nivel de confianza por debajo del umbral configurado
  Cuando se registra la afirmación
  Entonces queda marcada como pendiente de validación humana
  Y no se ofrece como valor del expediente mientras nadie la confirme
```

```gherkin
Escenario: Documento ilegible
  Dado un documento que el motor de extracción no puede procesar
  Cuando se ejecuta la extracción
  Entonces no se registra ninguna afirmación extraída
  Y el documento se marca para revisión humana con el motivo
  Y la ejecución fallida queda registrada igualmente
```

```gherkin
Escenario: La extracción es reintentable y no duplica
  Dado una extracción que falló por indisponibilidad del proveedor
  Cuando se reintenta
  Entonces no se producen afirmaciones duplicadas de la ejecución anterior
  Y cada intento queda registrado por separado
```

```gherkin
Escenario: Al modelo se le envía lo mínimo necesario
  Dado un expediente con varios documentos y datos personales
  Cuando se ejecuta la extracción sobre un documento
  Entonces solo se envía al proveedor el fragmento necesario para esa extracción
  Y queda registrado a qué proveedor se envió
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre lo extraído
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta afirmaciones extraídas con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Toda extracción produce **afirmaciones de origen `extraído`** (`HU-005`), nunca valores
  planos, y cada una cita su documento fuente y su ejecución de IA (`HU-018`).
- **La IA nunca sobrescribe en silencio.** No existe la operación de reemplazar una afirmación
  declarada por una extraída.
- Una afirmación extraída **no puede ascender a verificado**. Verificar exige fuente externa
  (Fase 3) o acción humana con respaldo.
- La extracción es un trabajo con estado persistido, reintentable e idempotente
  (`ADR-0001`): un reintento no duplica afirmaciones ni vuelve a cobrar la llamada.
- Se envía al proveedor **el fragmento necesario, no el expediente entero**, y queda registrado
  a dónde fueron esos datos (§33).
- La estrategia es híbrida: plantilla determinista donde el formato es conocido, modelo como
  respaldo para la cola larga. El motor vive detrás de un puerto reemplazable.
- Lo que el sistema no logra leer **no se rellena con una suposición**: se marca para revisión.

## Fuera de alcance

- La conciliación entre orígenes y su presentación → `HU-019`.
- La validación humana de lo extraído → `HU-020`.
- La verificación contra fuentes externas → Fase 3.
- La extracción de estructuras societarias y relaciones → Fase 4.
- La comparación de documentos entre sí y la detección de alteraciones → fase posterior.
- El registro detallado de la ejecución → `HU-018`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `afirmacion.origen` | Sí | Siempre `extraído` en esta historia | No |
| `afirmacion.evidencia_id` | Sí | Documento del que se extrajo | Sí |
| `afirmacion.confianza` | Sí | Entre 0 y 1, la que reporte el motor | No |
| `afirmacion.ejecucion_ia_id` | Sí | Ejecución que la produjo (`HU-018`) | No |
| `extraccion.ubicacion` | No | Página y zona del documento, si se puede determinar | No |
| `extraccion.estado` | Sí | `pendiente` \| `ejecutada` \| `fallida` \| `pendiente_validacion` | No |
| `extraccion.umbral_confianza` | Sí | Sale de la configuración `(TBD — PA-032)` | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: Fase 8, §2, §32, §33, §44 pregunta D
- Decisiones: `ADR-0005` (la IA es un origen), `ADR-0001` (estrategia híbrida, puerto
  reemplazable), `08-desarrollo/arquitectura-de-aplicacion.md` (minimización de datos enviados)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-032` — bloqueante: por debajo de qué confianza la extracción exige
  validación humana, y si lo fija cada organización cliente o hay un mínimo del producto.
  `PA-021` — qué proveedores de IA acepta el cliente. **Queda en `borrador`.**
- **Supuestos:** `SUP-006` (la IA propone; la persona firma), `SUP-004` (alojamiento en Estados
  Unidos, confirmado).
- **Depende de:** `HU-013` (hay documentos), `HU-005` (afirmaciones), `HU-018` (registro de
  ejecuciones).
- **Habilita a:** `HU-019`, `HU-020`.
- **Riesgo:** la tentación de "si la confianza es alta, dalo por bueno y autollena el campo". Es
  exactamente la sobrescritura silenciosa que el cliente prohíbe, y desde fuera se ve idéntica a
  un sistema que funciona bien.
