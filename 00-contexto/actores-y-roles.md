---
id: CTX-actores
estado: propuesto
actualizado: 2026-09-05
---

# Actores y roles

Derivado de la §4 y la §30 del
[documento funcional de Juan David](../01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md).
Cierra `PA-002`, `PA-013` y `PA-024`.

## Actores

| Actor | Qué es | Qué necesita de la plataforma |
|---|---|---|
| **Cliente del SaaS** | La empresa **obligada** que contrata la plataforma. No es una persona: es una organización con varios roles internos | Configurar su propio marco de cumplimiento y gestionar sus expedientes |
| **Contraparte** | La persona o empresa a la que se le hace debida diligencia: cliente, proveedor, contratista, empleado, accionista, conductor, transportadora aliada, intermediario, tercero pagador | Entregar su información y documentos sin fricción, y firmar |
| **Persona relacionada** | Alguien que "cuelga" de una contraparte: representante legal, beneficiario final, accionista, administrador, miembro de junta, apoderado, propietario o conductor de un vehículo | No interactúa necesariamente; puede requerir su propia diligencia según el motor de relaciones (Fase 10) |
| **Sistema** | Ejecuta lo automatizable: extracción, validaciones, consultas a fuentes, comparación de nombres, aplicación de reglas, cálculos preliminares, notificaciones, vencimientos, monitoreo | **Nunca decide** |
| **Visitante** | Persona sin cuenta que llega a la página de inicio pública (`EP-009`) desde un enlace o un buscador | Entender qué hace el producto y llegar al inicio de sesión si ya tiene acceso — nunca registro público (`PA-038`) |

## Roles dentro del cliente del SaaS

| Rol | Qué hace |
|---|---|
| **Administrador** | Configura la plataforma para su empresa: estándares, tipos de contraparte, matriz de requisitos, metodología de riesgo, fuentes. En la práctica lo ejerce el Oficial de Cumplimiento o el Representante Legal, y es quien **asigna roles y define la matriz de permisos** (`PA-024`) |
| **Oficial de Cumplimiento** | Responsable legal del proceso. Toma las decisiones finales y modifica la metodología |
| **Analista de Cumplimiento** | Revisa expedientes y alertas del día a día |
| **Revisor / Aprobador** | Según la política del cliente, aprueba o solo recomienda |
| **Auditor / Consulta** | Solo lectura, incluida la auditoría. Para revisiones internas o externas |
| **Usuario operativo** | Crea solicitudes de vinculación desde su área (comercial, compras, gestión humana) |

## Identidad, membresía y organización

Cierra `PA-013`. Un usuario **no pertenece** a una organización: tiene una identidad global y
una **membresía** por cada organización en la que trabaja, con roles y permisos distintos en
cada una.

```
Usuario (identidad global)
  └── Membresía ── Organización cliente ── Roles y permisos
  └── Membresía ── Organización cliente ── Roles y permisos
```

Es el caso del consultor o la firma que atiende a varios clientes, y también el del Oficial
de Cumplimiento que ejerce en varias empresas del mismo grupo — de ahí que los paquetes de
suscripción se construyan sobre esta figura (`PA-008`, `PA-015`).

Consecuencias que no son negociables:

- La interfaz tiene un **selector de organización**; toda pantalla opera en el contexto de una.
- **Nunca se autoriza por `user_id` solo.** La autorización es siempre `usuario × organización
  × acción`, evaluada contra la membresía activa.
- Cambiar de organización no arrastra permisos ni datos de la anterior (`HU-002`).

## Roles base y roles propios

Cierra `PA-024`. Los seis roles anteriores son **plantillas de permisos**, no un catálogo
cerrado:

- El administrador del tenant puede **crear roles propios** combinando permisos granulares.
- Los permisos se expresan por acción: `ver · crear · editar · aprobar · exportar · configurar
  · administrar`, sobre cada módulo.
- **"Rol" y "facultad de decisión" son cosas distintas.** Un rol da acceso; la facultad de
  aprobar se otorga aparte y respeta la segregación de funciones (`PA-034`: maker-checker
  configurable).
- El sistema impone límites duros que ningún rol personalizado puede saltarse: no
  autoaprobarse, no editar ni borrar la bitácora, no ver datos de otra organización.

El SaaS **no decide quién puede aprobar** dentro de la empresa del cliente. Solo garantiza
que lo que el cliente configuró se cumpla y quede registrado.

## Matriz de permisos base

Es un **punto de partida configurable**, no una definición fija: la §30 establece que cada
cliente ajusta la matriz según su propia segregación de funciones, y Juan David confirmó que
la ajusta él mismo (`PA-024`). Por eso el control de acceso es configuración versionada
(`ADR-0004`), no un `enum` en el código.

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
acceso propias — ver `08-desarrollo/arquitectura-de-aplicacion.md`. Juan David lo confirmó de
forma expresa: *"la contraparte usa un portal externo sin necesidad de cuenta completa ni
logeo"* (`PA-002`).

El enlace se lo puede enviar la plataforma por correo o copiarlo el usuario operativo para
hacerlo llegar por su cuenta; ambas vías están en alcance (`PA-031`, `HU-010`).
