---
id: DESC-supuestos
estado: vivo
actualizado: 2026-08-21
---

# Supuestos

Lo que asumimos mientras una `PA-xxx` sigue abierta. Un supuesto es una deuda: cuando la
pregunta se responde, se confirma o se derriba, y se corrige lo que dependía de él.

| ID | Supuesto | Sustituye a | Riesgo si es falso | Estado |
|----|----------|-------------|--------------------|--------|
| SUP-001 | El alcance inicial se limita a SARLAFT, SAGRILAFT y PTEE. | PA-001 | Alto — cambia el modelo de datos y el alcance del MVP | Vigente |
| SUP-002 | La plataforma es multi-entidad (varias organizaciones clientes en la misma instancia). | PA-008 | Alto — rehacer el modelo de acceso | **Confirmado** (§31) |
| SUP-003 | El contexto normativo es Colombia. | PA-001 | Medio | Vigente |
| SUP-004 | Los datos pueden alojarse en Estados Unidos. La SIC incluye a EE.UU. entre los países con nivel adecuado de protección (Circular Externa 5 de 2017). | — (confirmado por el cliente) | Bajo | Confirmado |
| SUP-005 | Todo es una organización: el usuario individual es una organización de un miembro. | PA-013 | Bajo — es la opción que mantiene ambas puertas abiertas | Vigente |
| SUP-006 | El agente de IA propone y justifica; la decisión siempre la firma una persona. | — (confirmado por el cliente) | Bajo | Confirmado |
| SUP-007 | El posicionamiento es "automatizamos y trazamos tu proceso de debida diligencia", no "hacemos tu SARLAFT". Es una decisión comercial y también de protección legal. | PA-022 | Medio | Vigente (§39) |
| SUP-008 | Las normas citadas por el cliente (circular de Supersociedades y resolución del sector transporte, ambas de 2026) **no están verificadas**. Se usan como contexto de diseño, nunca como hecho legal. | — | Alto si se comercializa sin revisión jurídica | Vigente (§47) |
