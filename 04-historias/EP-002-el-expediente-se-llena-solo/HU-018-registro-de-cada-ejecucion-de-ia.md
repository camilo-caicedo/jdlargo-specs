---
id: HU-018
titulo: Registro de cada ejecución de IA
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-08-27
---

# HU-018 — Registro de cada ejecución de IA

## Historia

**Como** Auditor
**quiero** poder ver qué modelo leyó qué documento, cuándo, con qué instrucciones y con qué
resultado
**para** poder explicar el origen de cualquier dato que la plataforma no recibió de una persona.

## Contexto

La §32 no deja margen: para cada uso de IA se registra modelo, proveedor, versión, fecha,
plantilla de instrucciones, documento fuente, resultado, nivel de confianza, quién lo validó y
el resultado final ya validado.

La misma sección enumera lo que **la IA nunca puede hacer**: inventar datos, completar campos
sin dejar constancia de que lo hizo ella, convertir una inferencia en un hecho confirmado,
eliminar una discrepancia, cerrar una alerta, aprobar o rechazar una vinculación, o modificar
un documento original. Esta historia es el mecanismo que permite demostrar que no lo hizo.

Hay un segundo motivo, menos evidente y más incómodo: usar un modelo alojado fuera del país
implica **enviar datos personales al exterior** (§33). La pregunta "¿a dónde fueron estos
datos?" tiene que tener respuesta, y la respuesta es este registro.

## Criterios de aceptación

```gherkin
Escenario: Registrar una ejecución completa
  Dado una extracción que se va a ejecutar sobre un documento
  Cuando el sistema invoca al proveedor de IA
  Entonces queda registrada la ejecución con proveedor, modelo, versión, plantilla de instrucciones, documento fuente, momento y resultado
  Y queda registrado el nivel de confianza reportado
  Y queda registrado a qué país o región se enviaron los datos
```

```gherkin
Escenario: Una ejecución fallida también se registra
  Dado un proveedor de IA que no responde o devuelve un error
  Cuando falla la ejecución
  Entonces queda registrada con su causa y su momento
  Y no se registra ninguna afirmación derivada de ella
```

```gherkin
Escenario: El registro es inmutable
  Dado una ejecución de IA ya registrada
  Cuando se intenta modificarla o eliminarla
  Entonces la operación es rechazada por la base de datos
  Y el registro permanece idéntico
```

```gherkin
Escenario: Toda afirmación extraída se puede rastrear hasta su ejecución
  Dado una afirmación de origen extraído
  Cuando se consulta su procedencia
  Entonces se obtiene la ejecución de IA que la produjo
  Y desde ella el modelo, el proveedor, la versión, la plantilla y el documento fuente
```

```gherkin
Escenario: Registrar quién validó el resultado
  Dado una afirmación extraída que una persona valida
  Cuando la validación ocurre
  Entonces la ejecución registra quién la validó, cuándo y cuál fue el resultado final
  Y ese registro no reemplaza el resultado original del modelo
```

```gherkin
Escenario: La IA no puede cerrar nada por su cuenta
  Dado una discrepancia abierta o una alerta pendiente
  Cuando una ejecución de IA produce un resultado que las explicaría
  Entonces la discrepancia o la alerta permanecen abiertas
  Y no existe ninguna operación por la que una ejecución de IA las cierre
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el registro de ejecuciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta las ejecuciones de IA con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Toda invocación a un proveedor de IA queda registrada, **haya salido bien o mal**, antes de que
  se derive ninguna afirmación de ella.
- El registro guarda: proveedor, modelo, versión, plantilla de instrucciones, documento fuente,
  fragmento enviado o su referencia, momento, resultado, confianza, destino de los datos, y
  quién validó el resultado final (§32).
- El registro es de **solo inserción**. Es parte de la bitácora en sentido amplio y comparte su
  regla (`HU-006`).
- Toda afirmación de origen `extraído` **cita** su ejecución. No existe una afirmación extraída
  huérfana.
- La IA no puede: inventar datos, completar sin constancia, convertir inferencia en hecho,
  eliminar una discrepancia, cerrar una alerta, aprobar o rechazar, ni modificar un documento
  original (§32). Cada una de esas prohibiciones se comprueba con una prueba automatizada.
- El proveedor vive detrás de un puerto intercambiable; cambiar de proveedor no cambia la forma
  del registro.

## Fuera de alcance

- La extracción en sí → `HU-017`.
- La validación humana → `HU-020`.
- La medición del costo de cada llamada al proveedor de IA y su facturación → Fase C.
- La evaluación de la calidad del modelo o su comparación con otros.
- El almacenamiento del fragmento enviado, si el contrato con el cliente no lo permite: se
  guarda su referencia y su huella, no necesariamente el contenido.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `ejecucion_ia.organization_id` | Sí | Organización cliente existente | No |
| `ejecucion_ia.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `ejecucion_ia.proveedor` | Sí | Proveedor del catálogo aceptado `(TBD — PA-021)` | No |
| `ejecucion_ia.modelo` y `version` | Sí | Texto no vacío | No |
| `ejecucion_ia.plantilla_instrucciones` | Sí | Identificador y versión de la plantilla usada | No |
| `ejecucion_ia.documento_id` | Condicional | Obligatorio si la ejecución partió de un documento | No |
| `ejecucion_ia.destino_datos` | Sí | País o región a la que se enviaron los datos | No |
| `ejecucion_ia.ocurrida_en` | Sí | Momento; se escribe una sola vez | No |
| `ejecucion_ia.resultado` | Sí | Resultado devuelto, o la causa del fallo | Sí |
| `ejecucion_ia.confianza` | Condicional | Obligatoria si hubo resultado | No |
| `ejecucion_ia.validada_por` | No | Usuario que validó el resultado final (`HU-020`) | No |
| `ejecucion_ia.resultado_final` | No | Lo que quedó tras la validación humana | Sí |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: §32, §33, §23, Fase 8
- Decisiones: `ADR-0005` (regla 4: la IA es un origen más, nunca una autoridad),
  `08-desarrollo/arquitectura-de-aplicacion.md` (puerto de IA, minimización)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-021` — bloqueante: qué proveedores de IA acepta el cliente y bajo
  qué contrato, dado que implica transferencia internacional de datos personales. **Queda en
  `borrador`.**
- **Supuestos:** `SUP-004` (alojamiento en Estados Unidos, confirmado), `SUP-006`.
- **Depende de:** `HU-006` (bitácora), `HU-002`.
- **Habilita a:** `HU-017`, `HU-020`, y toda ejecución de IA de las fases siguientes.
- **Riesgo:** este registro es lo que separa "usamos IA de forma responsable" de poder
  demostrarlo. Sin él, la política de uso de la §32 es una declaración de intenciones.
