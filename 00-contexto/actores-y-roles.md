---
id: CTX-actores
estado: propuesto
actualizado: 2026-08-25
---

# Actores y roles

Derivado de la §4 y la §30 del
[documento funcional del cliente](../01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md).
Cierra `PA-002`.

## Actores

| Actor | Qué es | Qué necesita de la plataforma |
|---|---|---|
| **Cliente del SaaS** | La empresa que contrata la plataforma. No es una persona: es una organización con varios roles internos | Configurar su propio marco de cumplimiento y gestionar sus expedientes |
| **Contraparte** | La persona o empresa a la que se le hace debida diligencia: cliente, proveedor, contratista, empleado, accionista, conductor, transportadora aliada, intermediario, tercero pagador | Entregar su información y documentos sin fricción, y firmar |
| **Persona relacionada** | Alguien que "cuelga" de una contraparte: representante legal, beneficiario final, accionista, administrador, miembro de junta, apoderado, propietario o conductor de un vehículo | No interactúa necesariamente; puede requerir su propia diligencia según el motor de relaciones (Fase 10) |
| **Sistema** | Ejecuta lo automatizable: extracción, validaciones, consultas a fuentes, comparación de nombres, aplicación de reglas, cálculos preliminares, notificaciones, vencimientos, monitoreo | **Nunca decide** |

## Roles dentro del cliente del SaaS

| Rol | Qué hace |
|---|---|
| **Administrador** | Configura la plataforma para su empresa: estándares, tipos de contraparte, matriz de requisitos, metodología de riesgo, fuentes |
| **Oficial de Cumplimiento** | Responsable legal del proceso. Toma las decisiones finales y modifica la metodología |
| **Analista de Cumplimiento** | Revisa expedientes y alertas del día a día |
| **Revisor / Aprobador** | Según la política del cliente, aprueba o solo recomienda |
| **Auditor / Consulta** | Solo lectura, incluida la auditoría. Para revisiones internas o externas |
| **Usuario operativo** | Crea solicitudes de vinculación desde su área (comercial, compras, gestión humana) |

## Matriz de permisos base

Es un **punto de partida configurable**, no una definición fija: la §30 establece que cada
cliente ajusta la matriz según su propia segregación de funciones. Por eso el control de
acceso es configuración versionada (`ADR-0004`), no un `enum` en el código.

| Función | Contraparte | Usuario operativo | Analista | Oficial de Cumplimiento | Auditor |
|---|:---:|:---:|:---:|:---:|:---:|
| Crear solicitud | ✓ | ✓ | ✓ | ✓ | — |
| Diligenciar y cargar documentos | ✓ | — | — | — | — |
| Revisar documentos | — | — | ✓ | ✓ | solo consulta |
| Resolver alerta | — | — | ✓ | ✓ | solo consulta |
| Modificar metodología | — | — | — | ✓ | solo consulta |
| Aprobar / Rechazar | — | — | según política | ✓ | — |
| Ver auditoría | — | — | ✓ | ✓ | ✓ |

## Nota de diseño

La contraparte **no es un usuario del sistema**: no tiene cuenta ni contraseña, entra con un
enlace y un token acotado a un solo expediente. Es una superficie distinta, con reglas de
acceso propias — ver `08-desarrollo/arquitectura-de-aplicacion.md`.
