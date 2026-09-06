---
id: DESC-supuestos
estado: vivo
actualizado: 2026-09-05
---

# Supuestos

Lo que asumimos mientras una `PA-xxx` sigue abierta. Un supuesto es una deuda: cuando la
pregunta se responde, se confirma o se derriba, y se corrige lo que dependía de él.

| ID | Supuesto | Sustituye a | Riesgo si es falso | Estado |
|----|----------|-------------|--------------------|--------|
| SUP-001 | El alcance inicial se limita a SARLAFT, SAGRILAFT y PTEE. | PA-001 | Alto — cambia el modelo de datos y el alcance del MVP | **Confirmado con matiz** — SARLAFT y SAGRILAFT son el núcleo; **PTEE entra como marco complementario**, no idéntico a los otros dos. Otros estándares después, sobre una capa activable. Ver `vision.md` |
| SUP-002 | La plataforma es multi-entidad (varias organizaciones clientes en la misma instancia). | PA-008 | Alto — rehacer el modelo de acceso | **Confirmado** (§31, `PA-008`) |
| SUP-003 | El contexto normativo es Colombia. | PA-001 | Medio | **Confirmado** — Supersociedades (SAGRILAFT/PTEE) y Supertransporte (SARLAFT sectorial) |
| SUP-004 | Los datos pueden alojarse en Estados Unidos. La SIC incluye a EE.UU. entre los países con nivel adecuado de protección (Circular Externa 5 de 2017). | — (confirmado por Juan David) | Bajo | Confirmado — pero la transferencia a un **proveedor de IA** exige además contrato/DPA revisado (`PA-021`, `PA-045`) |
| SUP-005 | Todo es una organización: el usuario individual es una organización de un miembro. | PA-013 | Bajo — es la opción que mantiene ambas puertas abiertas | **Derribado y reemplazado** — el modelo confirmado es `Usuario (identidad global) › Membresía › Organización › Roles`. La organización sigue siendo la unidad de aislamiento, pero la identidad vive por fuera de ella. Ver `actores-y-roles.md` |
| SUP-006 | El agente de IA propone y justifica; la decisión siempre la firma una persona. | — (confirmado por Juan David) | Bajo | **Confirmado y reforzado** (`PA-032`): la confianza de la IA es señal, no veredicto |
| SUP-007 | El posicionamiento es "automatizamos y trazamos tu proceso de debida diligencia", no "hacemos tu SARLAFT". Es una decisión comercial y también de protección legal. | PA-022 | Medio | **Confirmado** — formalizado en `ADR-0006` |
| SUP-008 | Las normas citadas por Juan David (circular de Supersociedades y resolución del sector transporte, ambas de 2026) **no están verificadas**. Se usan como contexto de diseño, nunca como hecho legal. | — | Alto si se comercializa sin revisión jurídica | **Vigente** (§47). Se extiende al Art. 28 de la Ley 962/2005 (retención de 10 años) y a la Ley 527/1999 (firma electrónica), citados al cerrar `PA-009` y `PA-020` |
| SUP-009 | El costo por consulta a fuentes colombianas está entre **$1.000 y $2.000 COP**. Cifra dada por Juan David, no cotizada. | PA-012 | **Alto** — es el insumo del modelo de precios. Si el costo real duplica esta cifra, los planes de `PA-043` nacen con margen negativo | Vigente — la investigación de `PA-040` (2026-09-05) encontró agregadores colombianos de KYC/LA-FT ya construidos (Tusdatos, Verdata, RiskTech, Compliance.com.co), pero ninguno publica precio; sigue sin cotización real |
| SUP-010 | Las fuentes colombianas admiten conexión directa entre la plataforma y la entidad, sin intermediario. | PA-005 | **Alto** — si no la admiten, entra un proveedor con su margen y su SLA, y cambia el costo y la arquitectura de integración | Parcialmente derribado — `PA-040` (2026-09-05) encontró una tercera vía que este supuesto no contemplaba: agregadores de dominio LA/FT ya construidos (Tusdatos, Verdata, RiskTech), ni conexión directa por fuente ni intermediario genérico. Falta decidir cuál de las tres vías se usa |
| SUP-011 | La aceptación electrónica con OTP por correo tiene valor probatorio suficiente para el MVP, sin firma digital certificada. | PA-020 | Medio-alto — si no lo tiene, hay que adelantar el módulo de firma certificada y contratar proveedor | Vigente — pendiente de revisión jurídica y de `PA-041` |
| SUP-012 | El primer despliegue se hace con **dos estándares (SARLAFT y PTEE)** y siete tipos de contraparte (conductor, propietario, poseedor, proveedor, cliente, empleado, accionista). | PA-018 | Bajo — el motor es genérico; esto es carga inicial, no especificación | **Confirmado** por Juan David |
