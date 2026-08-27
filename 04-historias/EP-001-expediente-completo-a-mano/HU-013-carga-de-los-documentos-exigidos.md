---
id: HU-013
titulo: Carga de los documentos exigidos
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-013 — Carga de los documentos exigidos

## Historia

**Como** Contraparte
**quiero** ver qué documentos me piden y cargarlos desde el mismo sitio donde diligencié mis
datos
**para** terminar de entregar lo que me corresponde sin que nadie tenga que perseguirme por
correo.

## Contexto

Es la Fase 7 del documento del cliente. La lista de documentos no está escrita en ninguna
pantalla: sale de la matriz de requisitos de la versión que cita el expediente (`HU-007`).

Cada documento cargado guarda tipo, emisor, fechas, el archivo, **una huella digital del
archivo** —para detectar si se alteró—, tamaño, formato, versión, estado y quién o qué lo
validó. La huella es lo que convierte al expediente en evidencia: sin ella, "este es el
documento que entregó" es una afirmación sin respaldo.

El archivo **no viaja por la aplicación**: se sube directamente al almacenamiento con una
dirección firmada, porque la plataforma de despliegue limita el tamaño de las peticiones a 4,5
MB (`ADR-0001`). Es un detalle de implementación con consecuencia funcional directa sobre el
tamaño de archivo aceptable.

En la Fase 1 el documento se recibe y se revisa. **Sus vigencias y su ciclo completo de estados
llegan en la Fase 2**, por decisión del roadmap.

## Criterios de aceptación

```gherkin
Escenario: Ver los documentos exigidos
  Dado un expediente de tipo "proveedor" que cita la versión 3 de configuración
  Cuando la contraparte abre la sección de documentos
  Entonces ve exactamente los tipos documentales que la matriz de esa versión exige a ese tipo
  Y cada uno indica si es obligatorio
  Y ve cuáles ya entregó y cuáles le faltan
```

```gherkin
Escenario: Cargar un documento y registrar su huella digital
  Dado un tipo documental exigido y todavía no entregado
  Cuando la contraparte carga un archivo válido
  Entonces el documento queda asociado al expediente y a ese tipo documental
  Y se registra su huella digital, tamaño, formato, fecha de carga y quién lo cargó
  Y el documento queda en estado recibido
  Y la carga queda registrada en la bitácora
```

```gherkin
Escenario: La huella digital detecta que el archivo cambió
  Dado un documento ya cargado con su huella digital registrada
  Cuando se recalcula la huella del archivo almacenado
  Entonces coincide con la registrada
  Y si no coincidiera, el documento se marca para revisión y se registra el hecho en la bitácora
```

```gherkin
Escenario: Reemplazar un documento conserva el anterior
  Dado un documento ya cargado para un tipo documental
  Cuando la contraparte carga otro archivo para ese mismo tipo documental
  Entonces queda como versión vigente el archivo nuevo
  Y la versión anterior se conserva completa, con su huella digital y su momento de carga
  Y el reemplazo queda registrado en la bitácora con quién lo hizo
```

```gherkin
Escenario: Un archivo que no cumple las condiciones es rechazado
  Dado un archivo con un formato no admitido o que excede el tamaño máximo
  Cuando la contraparte intenta cargarlo
  Entonces la carga es rechazada indicando el motivo
  Y no queda ningún documento a medias asociado al expediente
  Y lo que la contraparte ya había diligenciado se conserva intacto
```

```gherkin
Escenario: No se puede dar por completa la entrega con documentos obligatorios pendientes
  Dado un expediente con un tipo documental obligatorio sin entregar
  Cuando la contraparte intenta dar por terminada la entrega
  Entonces la operación es rechazada indicando qué falta
  Y el expediente no transita a "documentos recibidos"
```

```gherkin
Escenario: El archivo no es accesible sin autorización
  Dado un documento cargado en un expediente de "Alfa Ficticia S.A.S."
  Cuando alguien intenta descargarlo sin ser miembro de esa organización cliente ni portador del enlace de acceso de ese expediente
  Entonces la descarga es rechazada
  Y el intento queda registrado
```

## Reglas de negocio

- Los tipos documentales exigidos salen de la matriz de la versión citada por el expediente. No
  hay lista de documentos escrita en el programa.
- Todo documento guarda: tipo documental, emisor declarado, fechas declaradas de expedición y de
  vencimiento, archivo, **huella digital**, tamaño, formato, versión, estado, fuente y quién lo
  cargó (Fase 7).
- Las fechas y el emisor que informa la contraparte son **declarados** y se registran como
  afirmaciones con ese origen (`HU-005`). Comprobarlos contra el propio documento es Fase 2, y
  contra una fuente externa, Fase 3.
- **Un documento no se sobrescribe.** Cargar otro archivo para el mismo tipo documental crea una
  versión nueva y conserva la anterior íntegra.
- El archivo se sube directamente al almacenamiento con dirección firmada; no atraviesa la
  aplicación (`ADR-0001`).
- La descarga exige autorización: miembro de la organización cliente con permiso, o portador del
  enlace de acceso de ese mismo expediente (`HU-010`).
- El estado del documento en la Fase 1 se limita a `no recibido → recibido → en revisión →
  válido | requiere revisión | rechazado`. **El estado `vencido` y las vigencias llegan en la
  Fase 2.**
- Un documento rechazado se puede volver a cargar **sin perder lo diligenciado en el formulario**
  (§46).

## Fuera de alcance

- La extracción de datos del documento con IA → Fase 2.
- Las vigencias, el vencimiento y la renovación de documentos → Fase 2 y Fase 6.
- La validación automática de legibilidad o de que el documento sea el que dice ser → Fase 2.
- La verificación del documento contra una fuente externa → Fase 3.
- La firma electrónica sobre el documento → Fase 2.
- El cifrado del almacenamiento y la política de retención de archivos → depende de `PA-009`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `documento.organization_id` | Sí | Organización cliente existente | No |
| `documento.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `documento.tipo_documental` | Sí | Exigido por la matriz de la versión citada | No |
| `documento.version` | Sí | Correlativa por tipo documental dentro del expediente | No |
| `documento.huella` | Sí | Huella digital del archivo; se calcula al cargar y no se modifica | No |
| `documento.tamano` | Sí | Menor o igual al máximo configurado `(TBD — PA-030)` | No |
| `documento.formato` | Sí | Formato admitido `(TBD — PA-030)` | No |
| `documento.emisor_declarado` | No | Texto; se guarda como afirmación declarada | Sí |
| `documento.fecha_expedicion` / `fecha_vencimiento` | No | Fechas declaradas; se guardan como afirmaciones declaradas | Sí |
| `documento.estado` | Sí | `no_recibido` \| `recibido` \| `en_revision` \| `valido` \| `requiere_revision` \| `rechazado` | No |
| `documento.cargado_por` | Sí | Contraparte o usuario, con su tipo de actor | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 7, §46 (documento ilegible o rechazado), §44 pregunta C
- Decisiones: `ADR-0001` (subida directa con dirección firmada), `ADR-0005` (lo declarado sobre
  el documento es una afirmación)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-030` — bloqueante para las validaciones: qué formatos se admiten,
  qué tamaño máximo, y si un mismo tipo documental puede llegar repartido en varios archivos.
  `PA-009` — retención de los archivos. **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-007` (qué documentos se exigen), `HU-008`, `HU-010`, `HU-011`, `HU-005`,
  `HU-009`.
- **Habilita a:** `HU-014` (hay algo que revisar), y en la Fase 2 la extracción con IA.
- **Riesgo:** la huella digital tiene que calcularse en el momento de la carga y no después. Un
  documento almacenado sin huella no es evidencia, y no hay forma de otorgársela más tarde sin
  que la garantía se vuelva circular.
