---
id: PROD-capacidades
estado: propuesto
actualizado: 2026-09-05
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
| CAP-07 | Capa comercial | Planes, cupos, medición de consumo, facturación electrónica y cobro por link de pago (sin débito automático) | `EP-007` | Confirmada |
| CAP-08 | Salida de datos hacia otros sistemas | Entregar la información del expediente a otros sistemas del cliente, por API o por archivo estructurado (JSON/TXT configurable) | **Por abrir** | **Confirmada** (`PA-010`) — es el "MVP4" del cliente y no tiene épica todavía |

## Candidatas iniciales — resueltas

> `PA-001` a `PA-007` están cerradas (2026-09-05). Esta lista era la hipótesis de
> descubrimiento; queda su destino final, que es lo útil de conservar.

| Candidata inicial | Destino |
|---|---|
| Registro y administración de entidades / contrapartes | → `CAP-01` |
| Captura de información y documentos (formularios, cargue, vigencias) | → `CAP-01` y `CAP-02` |
| Consulta en fuentes externas y listas restrictivas | → `CAP-03` |
| Evaluación y calificación de riesgo | → `CAP-04` |
| Gestión de alertas y casos | → `CAP-03` |
| ~~Emisión y vigencia de certificaciones~~ | **Eliminada.** El producto no certifica (`ADR-0006`, `PA-003`, `PA-022`). Lo que queda es el **Informe de Debida Diligencia** y la **constancia de proceso ejecutado**, dentro de `CAP-06` |
| Reportes e informes | → `CAP-06` |
| Auditoría y trazabilidad | → `CAP-00` (es sustrato, no una capacidad aparte) |
| Administración de usuarios, roles y parametrización | → `CAP-05` |
