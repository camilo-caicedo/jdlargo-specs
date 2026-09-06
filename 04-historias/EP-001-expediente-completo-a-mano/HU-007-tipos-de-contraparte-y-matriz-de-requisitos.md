---
id: HU-007
titulo: Tipos de contraparte y matriz de requisitos
estado: en-revision
epica: EP-001
prioridad: Must
actualizado: 2026-09-06
---

# HU-007 — Tipos de contraparte y matriz de requisitos

> **Actualización 2026-09-05 (`PA-001`, `PA-017`, `PA-018`).** Modelo híbrido: **nosotros
> entregamos plantillas base** por régimen y sector, y el cliente las ajusta. El primer
> despliegue arranca con **dos estándares (SARLAFT y PTEE)** y siete tipos de contraparte:
> conductor, propietario, poseedor, proveedor, cliente, empleado y accionista. **PTEE es un marco
> complementario**, no un clon de SARLAFT: tiene requisitos propios y no debe modelarse igual.
> El motor sigue siendo genérico; esto es carga inicial, no especificación. Ver `ADR-0004` §6.

## Historia

**Como** Oficial de Cumplimiento
**quiero** que qué campos y qué documentos se le exigen a cada tipo de contraparte esté escrito
en la configuración de mi organización cliente
**para que** cambiar lo que se pide sea publicar una versión, y no pedirle al proveedor que
modifique el programa.

## Contexto

La Fase 2 del documento del cliente llama a la matriz de requisitos **"el módulo más
importante"**, y `ADR-0004` explica por qué: es lo que convierte el producto en un motor que
ejecuta reglas ajenas, en vez de una aplicación con el cumplimiento cosido por dentro.

`HU-004` construyó el envase —versiones inmutables con su puntero de configuración vigente—.
Esta historia mete dentro el primer contenido real: los tipos de contraparte y, para cada
combinación de estándar y tipo, qué campos y qué tipos documentales se exigen.

**En la Fase 1 esa configuración se carga a mano**, por operación de sistema. Es una decisión
deliberada del roadmap: mientras haya un solo cliente ancla, configurarlo a mano es más rápido
que construir la pantalla, que llega en la Fase 5. Lo que no se aplaza es que la configuración
sea **datos versionados** desde el primer expediente.

## Criterios de aceptación

```gherkin
Escenario: Cargar tipos de contraparte en una versión de configuración
  Dado una versión de configuración en borrador de "Alfa Ficticia S.A.S."
  Cuando se cargan los tipos de contraparte que esa organización cliente utiliza
  Y se publica la versión
  Entonces los tipos quedan disponibles para abrir solicitudes de vinculación
  Y ningún tipo de contraparte aparece escrito en el código del programa
```

```gherkin
Escenario: La matriz define qué se exige a cada combinación
  Dado una versión de configuración publicada con el estándar aplicable y el tipo de contraparte "proveedor"
  Cuando se consulta qué exige esa combinación
  Entonces se obtiene la lista de campos requeridos con su obligatoriedad y sus validaciones
  Y la lista de tipos documentales requeridos
  Y cada requisito indica de qué regla de la matriz proviene
```

```gherkin
Escenario: Dos tipos de contraparte exigen cosas distintas
  Dado una versión de configuración con los tipos "proveedor" y "conductor"
  Y una matriz que exige a "conductor" un tipo documental que no exige a "proveedor"
  Cuando se consulta qué exige cada tipo
  Entonces las dos listas son distintas
  Y la diferencia proviene de la matriz, no de una condición escrita en el programa
```

```gherkin
Escenario: Cambiar la matriz es publicar una versión nueva
  Dado una versión de configuración publicada cuya matriz no exige cierto documento a "proveedor"
  Cuando el Oficial de Cumplimiento publica una versión nueva que sí lo exige
  Entonces las solicitudes que se abran desde ese momento lo exigen
  Y la versión anterior sigue legible con la matriz que tenía
  Y no fue necesario desplegar una versión nueva del programa
```

```gherkin
Escenario: Una matriz incompleta no se puede publicar
  Dado una versión de configuración en borrador con un tipo de contraparte sin ningún requisito definido
  Cuando se intenta publicar la versión
  Entonces la publicación es rechazada indicando qué combinación quedó sin requisitos
  Y la versión permanece en borrador
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la matriz
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta los tipos de contraparte y la matriz de requisitos con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna fila de "Beta Ficticia S.A.S."
  Y un intento de escribir en la matriz de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Los tipos de contraparte y la matriz de requisitos son **contenido de una versión de
  configuración** (`HU-004`): se publican, no se editan.
- Una fila de la matriz asocia una combinación de **estándar × tipo de contraparte** con un
  requisito, que es un **campo** o un **tipo documental**, y con su obligatoriedad.
- El código conoce la forma de un requisito; nunca su contenido. Ninguna referencia a un
  estándar concreto, a un tipo de contraparte concreto o a un documento obligatorio concreto
  aparece literalmente en el programa.
- Los requisitos condicionales se expresan con el conjunto cerrado de condiciones de
  `ADR-0004`: campo, operador, valor y combinadores. **No se admite un lenguaje libre dentro de
  la configuración.**
- Una versión no se puede publicar si alguna combinación declarada queda sin ningún requisito:
  un formulario vacío no es una configuración válida.
- El sector transporte necesita su propia plantilla y no debe forzarse la genérica (Fase 6 del
  documento del cliente). Es un caso de uso de la matriz, no una excepción en el código.

## Fuera de alcance

- La interfaz para que el cliente administre la matriz → Fase 5. Aquí se carga por operación de
  sistema.
- El motor de riesgo, sus factores y sus umbrales → Fase 4.
- El catálogo de fuentes externas → Fase 3.
- El tratamiento de personas relacionadas y el motor de relaciones → Fase 4.
- La generación del formulario a partir de la matriz → `HU-012`. Aquí se define qué se exige;
  allí se arma la pantalla.
- Reglas de vigencia por tipo documental → Fase 2.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `counterparty_type.organization_id` | Sí | Organización cliente existente | No |
| `counterparty_type.configuration_version_id` | Sí | Versión existente (`HU-004`) | No |
| `counterparty_type.name` | Sí | Único dentro de la versión | No |
| `counterparty_type.nature` | Sí | `natural_person` \| `legal_entity` | No |
| `requirement.standard` | Sí | Estándar declarado en la versión `(por validar — PA-001)` | No |
| `requirement.counterparty_type_id` | Sí | Tipo de la misma versión | No |
| `requirement.type` | Sí | `field` \| `document_type` | No |
| `requirement.key` | Sí | Identificador del campo o tipo documental | No |
| `requirement.mandatory` | Sí | `always` \| `conditional` \| `optional` | No |
| `requirement.condition` | Condicional | Obligatoria si la obligatoriedad es `conditional`; solo condiciones del conjunto cerrado | No |
| `requirement.validation` | No | Tipo de dato, formato, rango; según el conjunto cerrado | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 1 (tipos de contraparte), Fase 2 (matriz de requisitos), Fase 6
  (plantilla de transporte), §41
- Decisiones: `ADR-0004`
- Modelo: `05-datos/modelo-conceptual.md`, plano de Configuración

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna. `PA-017`, `PA-018` y `PA-001` **resueltas** → `ADR-0004` §6 y
  `00-contexto/vision.md`.
- **Supuestos:** `SUP-001` (alcance SARLAFT / SAGRILAFT / PTEE), `SUP-008` (la norma del sector
  transporte que cita el cliente no está verificada: se soporta como plantilla configurable,
  no como regla afirmada).
- **Depende de:** `HU-004` (versiones de configuración), `HU-002` (aislamiento).
- **Habilita a:** `HU-008` (la solicitud elige un tipo), `HU-012` (el formulario se arma desde
  la matriz), `HU-013` (los documentos exigidos salen de la matriz).
- **Riesgo:** es la historia donde más tienta escribir "si el tipo es conductor, pedir licencia".
  Cada una de esas condiciones metida en el código es una bifurcación del producto por cliente,
  y es exactamente lo que `ADR-0004` prohíbe.
