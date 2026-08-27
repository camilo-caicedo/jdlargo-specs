---
id: HU-044
titulo: Exportación del expediente y de reportes
estado: borrador
epica: EP-006
prioridad: Should
actualizado: 2026-08-27
---

# HU-044 — Exportación del expediente y de reportes

## Historia

**Como** Auditor
**quiero** llevarme el expediente completo y los reportes en un archivo
**para** poder entregarlos a quien los pida sin darle acceso a la plataforma ni tener que
reconstruirlos a mano.

## Contexto

Es el punto 24 de la §35: exportación de reportes. Y es lo que le falta a `HU-016` para cerrar el
ciclo: allí el expediente se reconstruye en pantalla; aquí se puede sacar.

El uso real es concreto: una auditoría externa, un requerimiento, una revisión interna. En todos
esos casos lo que se entrega tiene que ser **fiel y completo**, y tiene que poder demostrarse que
no se manipuló después de generarse.

`ADR-0001` ya eligió las herramientas y dónde corren: generación en el servidor, en PDF y en hoja
de cálculo.

## Criterios de aceptación

```gherkin
Escenario: Exportar un expediente completo
  Dado un expediente cerrado
  Cuando el Auditor lo exporta
  Entonces obtiene un archivo con todo su recorrido: requisitos exigidos, información entregada con su procedencia, documentos, verificaciones, alertas, casos, evaluaciones y decisiones
  Y el archivo indica con qué versión de configuración se armó y evaluó el expediente
```

```gherkin
Escenario: La exportación es fiel a lo que muestra la plataforma
  Dado un expediente exportado
  Cuando se compara con la consulta en pantalla
  Entonces el contenido coincide
  Y ningún dato aparece sin su origen ni presentado como verificado si no lo está
```

```gherkin
Escenario: La exportación deja rastro
  Dado una exportación ejecutada
  Cuando se consulta la bitácora
  Entonces consta quién exportó, qué exportó, cuándo y desde dónde
```

```gherkin
Escenario: El archivo generado es comprobable
  Dado un archivo de exportación
  Cuando se consulta su registro
  Entonces existe la huella digital del archivo generado
  Y permite comprobar que el archivo entregado es el que produjo la plataforma
```

```gherkin
Escenario: Exportar un reporte de gestión
  Dado un periodo y unos filtros
  Cuando el Oficial de Cumplimiento exporta el reporte correspondiente
  Entonces obtiene una hoja de cálculo con los datos que su rol le permite ver
  Y el reporte indica qué filtros se aplicaron y en qué momento se generó
```

```gherkin
Escenario: La exportación respeta los permisos
  Dado un usuario cuyo rol no le permite ver ciertos expedientes
  Cuando exporta
  Entonces el archivo contiene únicamente lo que puede consultar
  Y no se incluye ningún dato al que no tenga acceso
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la exportación
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando exporta expedientes o reportes
  Entonces el archivo contiene únicamente datos de "Alfa Ficticia S.A.S."
```

```gherkin
Escenario: Los documentos originales se pueden incluir
  Dado un expediente con documentos cargados
  Cuando se exporta con los documentos incluidos
  Entonces el paquete contiene los archivos originales tal como se cargaron
  Y cada uno conserva su huella digital registrada
```

## Reglas de negocio

- La exportación es **fiel**: lo exportado coincide con lo consultable, y cada dato conserva su
  procedencia.
- Toda exportación queda **registrada en la bitácora** con quién, qué, cuándo y desde dónde. Sacar
  datos personales de la plataforma es un hecho auditable.
- Del archivo generado se registra su **huella digital**, de modo que se pueda comprobar que es el
  que produjo la plataforma.
- La exportación **respeta los permisos y el aislamiento**: nunca incluye lo que el usuario no
  podría ver en pantalla.
- Los reportes indican qué filtros se aplicaron y cuándo se generaron. Un reporte sin sus
  parámetros no es reproducible.
- La generación ocurre en el servidor (`ADR-0001`).
- Los documentos originales se incluyen tal como se cargaron, con su huella (`HU-013`).

## Fuera de alcance

- El envío automático de reportes por correo o su programación periódica.
- Formatos exigidos por una autoridad concreta: no hay definición y no se inventa.
- El portal para auditores externos → fase posterior (§36).
- Una interfaz de programación para que el cliente extraiga datos por su cuenta: `ADR-0001` la
  reserva para cuando exista un tercero que la consuma.
- La exportación masiva de toda la cartera en un solo archivo.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `exportacion.organization_id` | Sí | Organización cliente existente | No |
| `exportacion.tipo` | Sí | `expediente` \| `reporte` | No |
| `exportacion.alcance` | Sí | Expediente concreto, o filtros y periodo del reporte | No |
| `exportacion.formato` | Sí | `pdf` \| `hoja_de_calculo` \| `paquete_con_documentos` | No |
| `exportacion.solicitada_por` | Sí | Usuario con permiso de exportar | No |
| `exportacion.generada_en` | Sí | Momento; se escribe una sola vez | No |
| `exportacion.huella_archivo` | Sí | Huella digital del archivo generado | No |
| `exportacion.direccion_red` | Sí | Desde dónde se solicitó | Sí (dato personal indirecto) |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: §35 punto 24, Fase 18, Fase 22, §30
- Decisiones: `ADR-0001` (generación de PDF y hoja de cálculo en el servidor)
- Historias: completa `HU-016`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-007` (resuelta en cuanto a que hace falta exportación), `PA-009`
  (retención de lo exportado). Queda en `borrador` por arrastre.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-016` (el expediente reconstruible), `HU-043` (los reportes de gestión),
  `HU-003`.
- **Habilita a:** que una auditoría externa se atienda sin dar acceso a la plataforma.
- **Riesgo:** la exportación es la vía más directa por la que salen datos personales de la
  plataforma. Registrar cada exportación no es burocracia: es lo que permite responder quién se
  llevó qué, que es una pregunta que se acaba haciendo.
