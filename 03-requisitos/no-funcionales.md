---
id: REQ-no-funcionales
estado: borrador
actualizado: 2026-08-21
---

# Requisitos no funcionales

Un RNF sin número medible no es un RNF. "Debe ser rápido" no sirve; "p95 < 2 s con 200
usuarios concurrentes" sí.

| ID | Categoría | Requisito (medible) | Cómo se verifica | Estado |
|----|-----------|---------------------|------------------|--------|
| RNF-001 | Seguridad | _TBD_ | | Borrador |

## Categorías a cubrir

- **Seguridad y acceso** — autenticación, autorización por rol, cifrado en tránsito y reposo.
- **Protección de datos personales** — tratamiento de datos sensibles, consentimiento, retención. `(por validar — Ley 1581 de 2012)`
- **Auditoría y trazabilidad** — quién hizo qué y cuándo; inmutabilidad del registro. → `PA-009`
- **Disponibilidad y desempeño** — SLA, tiempos de respuesta, volúmenes esperados.
- **Retención y respaldo** — cuánto tiempo se conserva la evidencia. → `PA-009`
- **Multi-entidad / aislamiento de datos** — → `PA-008`, `SUP-002`
- **Usabilidad y accesibilidad**
- **Observabilidad y soporte**
- **Localización** — idioma español, zona horaria America/Bogota, formatos locales.
