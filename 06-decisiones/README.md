# Decisiones (ADR)

Una decisión con alternativas y consecuencias = un ADR. Archivo: `ADR-0001-titulo.md`.
Plantilla: `../_plantillas/adr.md`.

Un ADR **aceptado** no se edita cuando cambia la decisión: se escribe uno nuevo que lo
reemplaza, y el viejo pasa a estado `reemplazado` apuntando al nuevo. Mientras un ADR sigue en
`propuesto` sí se actualiza en sitio, dejando constancia con una nota de actualización al
inicio del documento.

| ID | Decisión | Fecha | Estado |
|----|----------|-------|--------|
| [ADR-0001](ADR-0001-stack-tecnico.md) | Stack técnico para el piloto y la primera versión comercial | 2026-08-21 | Propuesto |
| [ADR-0002](ADR-0002-pasarela-de-pagos-y-facturacion.md) | Cobro de suscripciones, medición de consumo y facturación electrónica | 2026-08-21 | Propuesto — actualizado 2026-09-05 |
| ADR-0003 | Auto-hospedaje del índice de screening (yente) | — | Aplazado |
| [ADR-0004](ADR-0004-reglas-como-datos.md) | El cumplimiento es configuración versionada, no código | 2026-08-25 | Propuesto — actualizado 2026-09-05 |
| [ADR-0005](ADR-0005-modelo-de-procedencia.md) | Todo dato del expediente lleva su procedencia | 2026-08-25 | Propuesto — actualizado 2026-09-05 |
| [ADR-0006](ADR-0006-posicionamiento-sin-certificacion.md) | El producto no certifica: automatiza y traza la debida diligencia | 2026-09-05 | Propuesto |
| [ADR-0007](ADR-0007-retencion-y-proteccion-de-la-bitacora.md) | Retención de la evidencia y nivel de protección de la bitácora | 2026-09-05 | Propuesto |
| [ADR-0008](ADR-0008-matching-multicriterio.md) | El matching es multicriterio y la confirmación es humana | 2026-09-05 | Propuesto |
| [ADR-0009](ADR-0009-monitoreo-continuo-y-control-de-costo.md) | Monitoreo continuo por riesgo y por evento, con control de costo | 2026-09-05 | Propuesto |
| [ADR-0010](ADR-0010-firma-y-evidencia-electronica.md) | Aceptación electrónica primero; firma certificada como módulo opcional | 2026-09-05 | Propuesto |
