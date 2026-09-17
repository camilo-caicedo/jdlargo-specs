---
id: HU-064
titulo: Firma física
estado: borrador
epica: EP-002
prioridad: Should
actualizado: 2026-09-17
---

# HU-064 — Firma física

> **Nota de alcance (2026-09-17).** Esta historia nace de prueba en vivo, no del documento del
> cliente. **No es el "nivel 3" de `HU-022`.** `HU-022`/`ADR-0010`/`PA-020` reservan "nivel 3"
> para firma digital certificada emitida por un proveedor externo acreditado, explícitamente
> diferida fuera del MVP. Esta historia es otra cosa, más simple: una contraparte que no puede o
> no quiere firmar electrónicamente firma en papel, y ese papel entra al expediente como
> evidencia. No requiere ningún proveedor externo ni cambia la decisión de `ADR-0010`.

## Historia

**Como** Analista de Cumplimiento
**quiero** generar un documento imprimible con lo que la contraparte debe firmar, y luego
adjuntar y registrar la firma física que me hace llegar de vuelta
**para** atender a las contrapartes que exigen o solo aceptan firma en papel, sin dejar el
expediente sin evidencia de conformidad.

## Contexto

`HU-022` construyó dos niveles de firma electrónica (aceptación con evidencia, y aceptación
reforzada con un factor adicional) y dejó explícito que el nivel 3 (firma digital certificada)
queda fuera del MVP por depender de un proveedor externo (`PA-020`). Ninguno de los dos resuelve
el caso real de una contraparte que, por política propia o por no tener acceso digital cómodo,
solo firma en papel.

La solución no es un proveedor de firma certificada: es dejar que el mismo contenido que hoy se
firmaría electrónicamente (la huella de lo que la contraparte revisó y aprobó, ver `HU-022`) se
entregue como PDF para imprimir, y que el papel firmado que regrese se trate como cualquier otro
documento del expediente (`HU-013`): se carga, se le calcula huella y formato real, pasa
antivirus, y queda asociado a esta modalidad de firma en vez de a un nivel electrónico.

Quien registra que la firma física llegó es el Analista de Cumplimiento, no la contraparte —
a diferencia de `HU-022`, aquí no hay ningún acto de la contraparte dentro de la plataforma más
allá de haber recibido el PDF fuera de ella.

## Criterios de aceptación

```gherkin
Escenario: Generar el documento para firma física
  Dado un expediente con datos declarados y extraídos ya conciliados, listo para firma
  Y una organización cliente configurada para admitir firma física
  Cuando el Analista de Cumplimiento pide generar el documento para firma física
  Entonces se genera un PDF con el contenido exacto que se firmaría electrónicamente
  Y el PDF queda con su propia huella digital, calculada al generarlo
  Y queda registrado en la bitácora quién lo generó y cuándo
```

```gherkin
Escenario: Adjuntar el documento firmado físicamente
  Dado un expediente con un documento de firma física ya generado
  Cuando el Analista de Cumplimiento adjunta el archivo escaneado o fotografiado del documento firmado
  Entonces el archivo pasa por las mismas validaciones que cualquier documento del expediente (antivirus, formato real, huella)
  Y queda asociado a la sección de firma del expediente, no a la de documentos exigidos por la matriz
  Y el expediente transita al estado siguiente, igual que con una firma electrónica
```

```gherkin
Escenario: Registrar la recepción de la firma física
  Dado un documento de firma física ya adjuntado
  Cuando el Analista de Cumplimiento confirma que la firma física fue recibida
  Entonces queda registrado quién la registró como recibida y cuándo
  Y la bitácora distingue explícitamente que la modalidad fue física, no electrónica de nivel 1 o 2
```

```gherkin
Escenario: El expediente distingue con qué modalidad se firmó
  Dado un expediente firmado con firma física
  Cuando se consulta cómo se firmó
  Entonces se indica que fue firma física, con quién la registró y cuándo
  Y no aparece como firma de nivel 1 o nivel 2
```

```gherkin
Escenario: Modificar lo firmado físicamente exige firmar de nuevo
  Dado un expediente ya firmado físicamente
  Cuando se registra una afirmación nueva sobre un campo cubierto por esa firma
  Entonces la firma física anterior se conserva con su documento y su momento
  Y el expediente exige una firma nueva, en cualquier modalidad admitida, antes de avanzar a decisión
```

```gherkin
Escenario: La firma física es configuración, no una vía siempre disponible
  Dado una organización cliente que no admite firma física en su configuración
  Cuando el Analista de Cumplimiento intenta generar el documento para firma física
  Entonces la operación es rechazada indicando que esa modalidad no está habilitada
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la firma física
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta documentos o registros de firma física con su contexto de usuario propagado
  Entonces obtiene únicamente los de expedientes de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- La firma física es una **modalidad adicional**, no un reemplazo de los niveles 1 y 2: la
  organización cliente la habilita o no en su configuración (`ADR-0004`), igual que el nivel de
  firma electrónica exigido.
- El PDF generado para firma física cubre **el mismo contenido concreto** que cubriría una firma
  electrónica del mismo expediente (`HU-022`): misma huella digital, misma versión aceptada.
- El documento firmado que regresa se valida exactamente igual que cualquier documento cargado al
  expediente (`HU-013`): antivirus, formato real, huella. Un archivo infectado o de formato
  inválido se rechaza sin llegar a asociarse a la firma.
- Registrar la recepción de la firma física es un acto humano explícito del Analista de
  Cumplimiento (o de quien tenga el permiso), nunca automático por el solo hecho de adjuntar el
  archivo.
- Si el contenido firmado cambia después, la firma física anterior **no vale** para el contenido
  nuevo: se conserva como evidencia, y hace falta una firma nueva (en cualquier modalidad
  admitida) antes de avanzar a decisión.
- La plataforma **no verifica forense ni caligráficamente** que la firma húmeda sea auténtica: se
  confía en el proceso operativo del cliente, igual que se confía en los datos declarados
  (`ADR-0005`).
- La plataforma **no afirma que la firma física sea jurídicamente suficiente** para un propósito
  determinado, en la misma línea que `HU-022` respecto a los niveles electrónicos.

## Fuera de alcance

- La firma digital certificada de proveedor externo ("nivel 3" de `HU-022`) — sigue siendo
  `ADR-0010`/`PA-020`, sin cambios; esta historia no la sustituye ni la adelanta.
- El envío físico o postal del documento impreso: ocurre fuera de la plataforma.
- La verificación forense o caligráfica de la firma húmeda.
- Que la contraparte tenga cualquier acto dentro de la plataforma para esta modalidad: aquí el
  único actor dentro del producto es el Analista de Cumplimiento.
- Firmar documentos por parte de usuarios internos del cliente (igual que en `HU-022`).

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `physical_signature.organization_id` | Sí | Organización cliente existente | No |
| `physical_signature.dossier_id` | Sí | Expediente de la misma organización cliente | No |
| `physical_signature.document_hash` | Sí | Huella digital del PDF generado para imprimir | No |
| `physical_signature.content_version` | Sí | Versión exacta del contenido que cubre esta firma | No |
| `physical_signature.generated_by` | Sí | Usuario que generó el documento imprimible | No |
| `physical_signature.generated_at` | Sí | Momento; se escribe una sola vez | No |
| `physical_signature.attached_document_id` | Condicional | Documento del expediente con el escaneo/foto firmado | No |
| `physical_signature.received_by` | Condicional | Usuario que registró la recepción; obligatorio para marcar recibida | No |
| `physical_signature.received_at` | Condicional | Momento del registro de recepción | No |
| `physical_signature.status` | Sí | `generated` \| `received` \| `invalid` (si el contenido cambió después) | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Historias: extiende `HU-022` con una tercera modalidad; reutiliza la carga de documentos de
  `HU-013`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna nueva. No depende de `PA-020` ni de `PA-041` (esas siguen
  siendo del nivel 3 certificado, no de esta historia).
- **Depende de:** `HU-022` (contenido y huella a firmar), `HU-013` (carga y validación del
  documento firmado), `HU-009` (transición de estado del expediente).
- **Habilita a:** que un cliente con contrapartes que exigen papel pueda cerrar expedientes sin
  bloquearse esperando una firma electrónica que la contraparte no va a dar.
- **Riesgo:** si la interfaz no distingue con claridad "firma física" de "firma electrónica de
  nivel 1/2" en cada lugar donde se muestra cómo se firmó un expediente, se pierde exactamente
  la trazabilidad que esta historia existe para dar.
