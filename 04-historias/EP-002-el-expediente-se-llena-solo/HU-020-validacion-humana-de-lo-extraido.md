---
id: HU-020
titulo: Validación humana de lo extraído
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-08-27
---

# HU-020 — Validación humana de lo extraído

## Historia

**Como** Analista de Cumplimiento
**quiero** confirmar o descartar lo que propuso la IA, campo por campo
**para** que ningún dato del expediente valga por el hecho de que una máquina lo leyó.

## Contexto

`SUP-006` lo fija como principio del proyecto y la §32 como prohibición explícita: la IA
propone y justifica; la persona confirma. Un dato extraído es una propuesta con confianza
asociada, y sigue siéndolo hasta que alguien con nombre y cargo la acepte.

La distinción que hay que sostener aquí es fina y es la de la §2: **validar no es verificar**.
Que un analista confirme que la IA leyó bien lo que dice el documento no convierte el dato en
verificado; solo confirma que el documento dice eso. Verificado es lo que confirma una fuente
externa autorizada, y eso es la Fase 3.

## Criterios de aceptación

```gherkin
Escenario: Confirmar un dato extraído
  Dado una afirmación extraída pendiente de validación
  Cuando el Analista de Cumplimiento la confirma
  Entonces queda registrado quién la validó y cuándo
  Y la afirmación pasa a estar disponible como dato del expediente
  Y su origen sigue siendo extraído, no verificado
```

```gherkin
Escenario: Descartar un dato extraído
  Dado una afirmación extraída que no corresponde a lo que dice el documento
  Cuando el Analista de Cumplimiento la descarta indicando el motivo
  Entonces la afirmación queda en estado descartada, sin borrarse
  Y queda registrado quién la descartó, cuándo y por qué
  Y el campo vuelve a quedar pendiente si la matriz lo exige
```

```gherkin
Escenario: Corregir un dato extraído crea una afirmación nueva
  Dado una afirmación extraída con un valor equivocado
  Cuando el Analista de Cumplimiento escribe el valor correcto tomándolo del documento
  Entonces se registra una afirmación nueva atribuida a esa persona, con su momento
  Y la afirmación extraída original se conserva descartada
  Y ambas quedan visibles en la conciliación
```

```gherkin
Escenario: Lo validado no asciende a verificado
  Dado una afirmación extraída ya confirmada por una persona
  Cuando se consulta su origen
  Entonces sigue siendo extraído, con la marca de haber sido validada y por quién
  Y no existe ninguna operación que la convierta en verificada sin una fuente externa
```

```gherkin
Escenario: Un dato de baja confianza no se usa sin validar
  Dado una afirmación extraída con confianza por debajo del umbral configurado
  Cuando se intenta usarla como dato del expediente sin validación
  Entonces la operación es rechazada
  Y el campo aparece como pendiente de validación
```

```gherkin
Escenario: Validar exige permiso
  Dado un usuario cuyo rol vigente no incluye el permiso de validar extracciones
  Cuando intenta confirmar una afirmación extraída
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la validación
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando intenta validar una afirmación de un expediente de "Beta Ficticia S.A.S."
  Entonces la operación es rechazada por la política de la base de datos
```

## Reglas de negocio

- Toda afirmación de origen `extraído` nace **pendiente de validación** si su confianza está por
  debajo del umbral configurado; por encima, la configuración decide si se valida igualmente.
- **Validar no cambia el origen.** Una afirmación extraída y validada sigue siendo extraída, con
  la marca de quién la validó (§2).
- Descartar no borra: la afirmación queda en estado `descartada` y sigue siendo evidencia.
- Corregir a mano produce una afirmación nueva atribuida a la persona que la escribió, con su
  momento. No se edita la que produjo el modelo.
- La validación queda registrada también en la ejecución de IA correspondiente, como "resultado
  final ya validado" (§32).
- Validar requiere permiso explícito (`HU-003`).
- Ninguna validación es automática, ni siquiera con confianza máxima. El umbral decide qué
  necesita revisión **obligatoria**, no qué se aprueba solo.

## Fuera de alcance

- La verificación contra fuentes externas → Fase 3.
- La resolución de discrepancias entre orígenes → `HU-019`. Validar una extracción y resolver
  una discrepancia son dos actos distintos, aunque a menudo ocurran seguidos.
- La revisión por parte de la contraparte del formulario autollenado → `HU-022`, que es donde
  ella corrige y firma.
- La medición de la calidad del modelo a partir de cuántas validaciones se descartan.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `validacion.organization_id` | Sí | Organización cliente existente | No |
| `validacion.afirmacion_id` | Sí | Afirmación de origen `extraído` | No |
| `validacion.resultado` | Sí | `confirmada` \| `descartada` \| `corregida` | No |
| `validacion.motivo` | Condicional | Obligatorio si se descarta o se corrige | No |
| `validacion.validada_por` | Sí | Usuario con permiso de validar | No |
| `validacion.validada_en` | Sí | Momento; se escribe una sola vez | No |
| `afirmacion.estado` | Sí | `pendiente_validacion` \| `vigente` \| `descartada` | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: §2, §32, Fase 8, Fase 17
- Decisiones: `ADR-0005` (una afirmación extraída no asciende a verificada sin respaldo)
- Supuesto: `SUP-006`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-032` — bloqueante: qué umbral obliga a validación. **Queda en
  `borrador`.**
- **Supuestos:** `SUP-006` (la IA propone; la persona firma).
- **Depende de:** `HU-017`, `HU-018`, `HU-005`, `HU-003`.
- **Habilita a:** `HU-019` (una discrepancia entre datos ya validados es más informativa), y la
  decisión de la Fase 1 sobre expedientes autollenados.
- **Riesgo:** con volumen alto, validar campo por campo se vuelve tedioso y aparece la presión
  de "validar todo lo que venga con confianza alta de un clic". Si esa función se construye, deja
  de haber validación humana y solo queda su apariencia.
