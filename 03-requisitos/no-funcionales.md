---
id: REQ-no-funcionales
estado: propuesto
actualizado: 2026-09-05
---

# Requisitos no funcionales

Un RNF sin número medible no es un RNF. "Debe ser rápido" no sirve; "p95 < 2 s con 200
usuarios concurrentes" sí.

> Actualizado con las respuestas del cliente a `PA-008`, `PA-009`, `PA-014`, `PA-021`,
> `PA-026` y `PA-030`. Los que siguen en `Borrador` no tienen todavía un número que el
> cliente o una medición respalden; no se inventa uno.

## Aislamiento y seguridad

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-001 | Ninguna consulta puede devolver una fila de otra organización cliente. Toda tabla del dominio lleva `organization_id` y política RLS activa, sin excepciones | Prueba automatizada por tabla que intenta el acceso cruzado y debe fallar; se ejecuta en cada despliegue (`HU-002`) | Propuesto |
| RNF-002 | La autorización se evalúa siempre como `usuario × organización × acción`. Ninguna ruta autoriza solo por `user_id` | Revisión de código en cada handler + prueba de contexto de organización cruzado | Propuesto |
| RNF-003 | Archivos, claves, configuración y bitácora están aislados por organización cliente. No se reutiliza ningún expediente entre clientes | Auditoría de rutas de almacenamiento y prueba de acceso cruzado a Storage | Propuesto |
| RNF-004 | Cifrado en tránsito (TLS 1.2+) y en reposo para base de datos y almacenamiento de documentos | Configuración verificada del proveedor + escaneo periódico | Propuesto |
| RNF-005 | Todo archivo cargado pasa por antivirus, validación de MIME real (no solo extensión) y cálculo de hash antes de quedar disponible | Prueba con archivo infectado de laboratorio y con extensión falsificada | Propuesto |

## Auditoría y trazabilidad

Detalle en `ADR-0007`.

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-006 | La aplicación no ejecuta `UPDATE` ni `DELETE` sobre la bitácora. El rol de base de datos de la aplicación solo tiene `INSERT` y `SELECT` sobre ella | Revisión de *grants* + prueba que intenta actualizar y borrar, y debe fallar en el motor | Propuesto |
| RNF-007 | Cada entrada de bitácora se escribe en la misma transacción que el cambio que describe: no hay cambio sin rastro ni rastro sin cambio | Prueba de fallo inyectado a mitad de transacción | Propuesto |
| RNF-008 | Cada entrada lleva hash del evento desde el día uno, para permitir encadenamiento o WORM más adelante sin migrar la bitácora | Verificación de esquema + recálculo de hash sobre una muestra | Propuesto |
| RNF-009 | Sobre cualquier expediente cerrado se pueden responder las 16 preguntas de la §44 sin construir un reporte a propósito | Ejercicio de auditoría sobre un expediente al azar al cierre de cada fase | Propuesto |
| RNF-010 | Un intento rechazado por permiso o por aislamiento también se registra | Prueba de acceso denegado que verifica la entrada en bitácora | Propuesto |

## Retención y respaldo

Detalle y reparto de responsabilidad en `ADR-0007`; decisión pendiente en `PA-044`.

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-011 | La retención del expediente de debida diligencia es configurable por organización cliente. Valor de referencia: **10 años** `(por validar — Art. 28 Ley 962/2005)` | Revisión de la política aplicada por tenant + contrato firmado | Propuesto |
| RNF-012 | La plataforma envía copias periódicas de la información al contacto que designe cada cliente, y registra el envío. Un envío fallido genera alerta: un respaldo delegado que falla en silencio no es un respaldo | Registro de envíos + prueba de fallo simulado | Propuesto |
| RNF-013 | La bitácora no se depura mientras la política de retención de esa organización no lo autorice explícitamente | Revisión de procesos programados de purga | Propuesto |
| RNF-014 | Objetivos de recuperación (RPO/RTO) | _TBD — sin número acordado_ | Borrador |

## Protección de datos personales

`(por validar — Ley 1581 de 2012)`

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-015 | Todo consentimiento se registra con la versión exacta del aviso aceptado, fecha, hora y medio | Inspección del registro de consentimiento (`HU-011`) | Propuesto |
| RNF-016 | El proveedor de IA es configurable por organización cliente y la IA se puede **desactivar por completo** para tenants sensibles | Prueba de tenant con IA desactivada: ningún dato personal sale hacia el proveedor | Propuesto |
| RNF-017 | Antes de enviar datos a un modelo externo se aplica minimización y, cuando sea viable, pseudonimización o redacción de campos sensibles | Revisión de la carga enviada en una muestra de ejecuciones de IA | Propuesto |
| RNF-018 | Toda transferencia internacional está amparada por contrato o DPA revisado, con registro de subencargados | Expediente contractual del proveedor (`PA-045`, `SUP-004`) | Propuesto |

## Volumen y desempeño

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-019 | El sistema soporta **10.000–50.000 contrapartes por organización cliente sin rediseño**, con indexación por organización, colas asíncronas, almacenamiento de documentos separado y búsqueda optimizada | Prueba de carga con datos ficticios en el extremo superior del rango (`PA-014`) | Propuesto |
| RNF-020 | Carga de referencia del primer cliente: **~1.000 consultas/mes** a fuentes externas | Medición real sobre la tabla de eventos de consumo | Propuesto |
| RNF-021 | Tiempos de respuesta (p95) por pantalla y por operación | _TBD — se fija al cierre de la Fase 1 con uso real_ | Borrador |
| RNF-022 | Disponibilidad y SLA comprometido por plan | _TBD — depende de `PA-043`_ | Borrador |

## Documentos

| ID | Requisito (medible) | Cómo se verifica | Estado |
|----|---------------------|------------------|--------|
| RNF-023 | Formatos aceptados: **PDF, JPG/JPEG y PNG** como núcleo; DOCX/XLSX según el caso | Prueba de carga por formato, incluida la de un formato no permitido | Propuesto |
| RNF-024 | Tamaño máximo **20 MB por archivo** como valor inicial, configurable por organización cliente y por plan dentro del rango 15–25 MB, sujeto a pruebas de infraestructura | Prueba en el límite y por encima del límite (`PA-030`) | Propuesto |
| RNF-025 | Un mismo tipo documental admite **varios archivos**, con deduplicación por hash | Prueba de carga múltiple y de archivo repetido | Propuesto |

## Pendientes

- **Usabilidad y accesibilidad** — sin criterio acordado todavía.
- **Observabilidad y soporte** — sin criterio acordado todavía.
- **Localización** — idioma español, zona horaria `America/Bogota`, formatos locales de fecha,
  número y moneda (COP). Falta convertirlo en verificación concreta.
