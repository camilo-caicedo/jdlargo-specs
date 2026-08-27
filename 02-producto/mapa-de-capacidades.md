---
id: PROD-capacidades
estado: borrador
actualizado: 2026-08-27
---

# Mapa de capacidades

Una capacidad es "algo que la plataforma sabe hacer", independiente de la pantalla que lo
implemente. Cada capacidad estable se convierte en una épica (`EP-xxx`) y de ahí salen las
historias.

| ID | Capacidad | Descripción | Épica | Estado |
|----|-----------|-------------|-------|--------|
| CAP-00 | Sustrato de cumplimiento auditable | Aislamiento entre organizaciones, procedencia del dato, configuración versionada y bitácora. No es visible para el usuario: es la condición para que todo lo demás sea auditable | `EP-000` | Confirmada |
| CAP-01 | Expediente de debida diligencia | Abrir, diligenciar, revisar, decidir y reconstruir un expediente completo sobre una contraparte | `EP-001` | Confirmada |
| CAP-02 | Extracción y conciliación | Leer los documentos con IA, contrastar lo extraído con lo declarado y hacer visible cada diferencia | `EP-002` | Confirmada |
| CAP-03 | Verificación y screening | Consultar fuentes externas y listas, producir coincidencias y convertir en casos las que exigen análisis | `EP-003` | Confirmada |
| CAP-04 | Relaciones y calificación de riesgo | Grafo de relaciones y beneficiario final, metodología de riesgo configurable y debida diligencia intensificada | `EP-004` | Confirmada |
| CAP-05 | Autogestión de la configuración | Que el cliente administre por sí mismo estándares, matriz, metodología, fuentes y permisos | `EP-005` | Confirmada |
| CAP-06 | Monitoreo y vida del expediente | Vigilar cambios después de la vinculación, renovar, vencer y dar visibilidad al Oficial de Cumplimiento | `EP-006` | Confirmada |
| CAP-07 | Capa comercial | Planes, cupos, medición de consumo, cobro y facturación electrónica | `EP-007` | Confirmada |

## Candidatas iniciales (hipótesis a validar con el cliente)

> Ninguna de estas está confirmada. Sirven solo para estructurar la conversación de
> descubrimiento; se confirman, se parten o se eliminan tras responder `PA-001` a `PA-007`.

- Registro y administración de entidades / contrapartes
- Captura de información y documentos (formularios, cargue, vigencias)
- Consulta en fuentes externas y listas restrictivas
- Evaluación y calificación de riesgo
- Gestión de alertas y casos
- Emisión y vigencia de certificaciones
- Reportes e informes
- Auditoría y trazabilidad
- Administración de usuarios, roles y parametrización
