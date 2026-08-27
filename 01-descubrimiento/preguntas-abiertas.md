---
id: DESC-preguntas
estado: vivo
actualizado: 2026-08-27
---

# Preguntas abiertas

Todo lo que no sabemos y bloquea una definición. Cuando una pregunta se responde:
se marca `Resuelta`, se anota la respuesta y se actualizan los documentos que dependían
de ella (y el `SUP-xxx` que la sustituía, si lo había).

| ID | Pregunta | Para quién | Bloquea | Estado | Respuesta |
|----|----------|-----------|---------|--------|-----------|
| PA-001 | ¿Cuál es el alcance regulatorio definitivo? ¿Solo SARLAFT, SAGRILAFT y PTEE, o hay más marcos? | Cliente | `vision.md`, `normativa/` | Abierta | — |
| PA-002 | ¿Quién usa la plataforma (roles) y quién la contrata? | Cliente | `actores-y-roles.md` | **Resuelta** | §4 y §30 del documento del cliente. Ver `00-contexto/actores-y-roles.md` |
| PA-003 | ¿Qué significa exactamente "certificar" una entidad? ¿Qué se emite, con qué vigencia y quién lo respalda? | Cliente | `glosario.md`, flujo principal | **Resuelta parcialmente** | El documento no usa "certificación": el producto automatiza y traza la debida diligencia, no certifica. Ver `PA-022` |
| PA-004 | ¿La plataforma evalúa a la entidad contratante, a sus contrapartes/terceros, o ambas? | Cliente | Modelo de datos, flujos | **Resuelta** | Se evalúa a las **contrapartes** del cliente (proveedores, clientes, conductores, empleados…) y a sus **personas relacionadas** según el motor de relaciones (Fase 10) |
| PA-005 | ¿Qué fuentes externas se consultan (listas restrictivas, registros públicos) y con qué proveedor? | Cliente | `07-integraciones/` | Abierta | — |
| PA-006 | ¿Hay una metodología de calificación de riesgo definida o hay que diseñarla? | Cliente | Motor de riesgo | **Resuelta** | No hay metodología única: **cada cliente configura la suya** (factores, pesos, umbrales). Ver `ADR-0004` |
| PA-007 | ¿Qué reportes / salidas necesita el usuario (informes, certificados, reportes a autoridad)? | Cliente | `02-producto/` | **Resuelta** | Expediente electrónico reconstruible, panel del Oficial de Cumplimiento (Fase 22) y exportación de reportes |
| PA-008 | ¿Multi-entidad / multi-tenant desde el día uno? | Cliente | Arquitectura | **Resuelta** | Multiempresa desde el día uno, con aislamiento estricto y prohibición de reutilizar datos entre clientes (§31) |
| PA-009 | ¿Requisitos de retención, auditoría y trazabilidad de la información? | Cliente | `03-requisitos/no-funcionales.md` | Abierta | — |
| PA-010 | ¿Hay un sistema o proceso actual que esto reemplaza? ¿Migración de datos? | Cliente | Alcance, roadmap | Abierta | — |
| PA-011 | ¿El producto ofrece **monitoreo continuo** (re-escaneo periódico de las contrapartes ya cargadas) o solo consulta puntual? Es la diferencia entre un negocio por consulta y uno por suscripción, y la principal fuente de costo variable. | Cliente | Modelo de precios, `ADR-0001`, modelo de datos | **Resuelta** | **Sí, el monitoreo continuo está en el alcance** (§35). Reactiva el riesgo de costo descrito en `07-integraciones/` |
| PA-012 | ¿Cuánto cobra por consulta un proveedor local de fuentes colombianas (Procuraduría, Contraloría, Policía, RUES)? Cotizar Tusdatos.co, Datacrédito Experian y Compliance.com.co. | Camilo (cotizar) | Modelo de precios, `07-integraciones/` | Abierta | — |
| PA-013 | ¿Un usuario puede pertenecer a **varias** organizaciones a la vez (p. ej. un consultor que atiende a varios clientes)? | Cliente | Modelo de identidad, `ADR-0001` | **Resuelta** | Sí: los roles son por organización y la matriz de permisos es configurable por cliente (§30) |
| PA-014 | ¿Cuántas contrapartes maneja un cliente típico? Es el número que determina el costo real por cliente y la viabilidad de cada plan. | Cliente | Modelo de precios, cuotas | Abierta | — |
| PA-015 | ¿Qué es una "licencia" en los planes: un asiento de usuario, un cupo de consultas, o ambos? | Cliente | Facturación, modelo de datos | **Resuelta** | La licencia incluye un **cupo de consultas mensuales**. Si el cliente lo supera, sube de plan o paga por consulta individual. Objetivo: controlar precio, gasto y margen. → `ADR-0002` |
| PA-016 | ¿Wompi permite cobro desatendido con tarjetas **Visa**, o el protocolo 3RI solo aplica a Mastercard? Si solo es Mastercard, una parte grande de los clientes pequeños no es cobrable automáticamente. | Wompi / Bancolombia | `ADR-0002`, módulo de cobro | Abierta — **bloqueante** | — |
| PA-017 | ¿Quién configura la matriz de requisitos en la práctica: el cliente por su cuenta o nosotros como servicio de implementación? Cambia por completo el esfuerzo de la interfaz de administración. | Cliente | `ADR-0004`, alcance del MVP | Abierta — **alto impacto** | — |
| PA-018 | ¿Cuántos estándares y tipos de contraparte hay que soportar en el primer cliente? Define si la configuración inicial se carga a mano o necesita interfaz desde el día uno. | Cliente | Alcance del MVP | Abierta | — |
| PA-019 | ¿El OTP por SMS es necesario o basta por correo? El SMS implica proveedor y costo por mensaje no presupuestado. | Cliente | Portal de contraparte | Abierta | — |
| PA-020 | ¿Qué proveedor de firma digital certificada (nivel 3) se usará, y desde qué fase? | Cliente | Fase 19 | Abierta | — |
| PA-021 | ¿Qué proveedores de IA acepta el cliente, considerando que implica transferencia internacional de datos personales (§33)? | Cliente | `ADR-0001`, contrato | Abierta | — |
| PA-022 | El nombre del proyecto habla de "certificación" pero el documento del cliente evita ese término (§39). ¿Se ajusta el posicionamiento del producto? | Cliente | Nombre, marketing, contrato | Abierta | — |
| PA-023 | El alcance del MVP (§35) son 24 módulos, muy por encima de las 20 semanas planificadas. ¿Se adopta la escalera de MVPs de la §43 y cuál es la fecha real objetivo? | Cliente + Camilo | **Todo el plan** | **Resuelta** | El alcance de la §35 se mantiene como MVP, pero se entrega **por fases de corte vertical**. Ver `02-producto/roadmap.md` |
| PA-024 | ¿El cliente puede crear roles propios dentro de su organización, o solo ajustar la matriz de permisos de los seis roles ya definidos? ¿Y quién asigna roles a los miembros? | Cliente | `HU-001`, `HU-003`, `actores-y-roles.md` | Abierta | — |
| PA-025 | Al publicar una versión nueva de configuración, ¿los expedientes en curso siguen con la versión anterior o migran a la nueva? Cambia el modelo: o el expediente congela la versión al abrirse, o la congela al evaluarse. | Cliente | `HU-004`, `ADR-0004`, modelo de datos | Abierta — **alto impacto** | — |
| PA-026 | La §23 pide una bitácora "inmutable **o** técnicamente protegida". ¿Basta con solo inserción por permisos de base de datos, o se exige encadenamiento criptográfico / almacenamiento WORM? No se puede añadir después sin rehacer la bitácora. | Cliente + asesoría jurídica | `HU-006`, `RNF` de auditoría | Abierta — **alto impacto** | — |
| PA-027 | ¿Cuál es la regla de precedencia por defecto entre afirmaciones del mismo campo (¿verificado > declarado > extraído?) y puede cada cliente cambiarla? | Cliente | `HU-005`, conciliación (Fase 2) | Abierta | — |
| PA-028 | La contraparte no completa el proceso y el enlace de acceso expira. ¿El expediente se cierra, se emite un enlace nuevo automáticamente, o queda a la espera de que el responsable interno lo reactive? La §46 señala el caso pero no lo resuelve. | Cliente | `HU-011`, `HU-013` | Abierta | — |
| PA-029 | ¿Puede el Oficial de Cumplimiento decidir sobre un expediente que todavía tiene requisitos pendientes, dejándolo registrado como excepción, o el sistema debe impedirlo? | Cliente | `HU-017`, máquina de estados | Abierta — **alto impacto** | — |
| PA-030 | ¿Qué formatos de archivo y qué tamaño máximo por documento se aceptan? ¿Se admite que un mismo tipo documental llegue en varios archivos? | Cliente | `HU-015` | Abierta | — |
| PA-031 | ¿Quién entrega el enlace de acceso a la contraparte: lo envía la plataforma por correo, o lo copia el usuario operativo y lo hace llegar por su cuenta? Define si la Fase 1 necesita ya notificaciones automáticas. | Cliente | `HU-010`, `HU-011` | Abierta | — |
