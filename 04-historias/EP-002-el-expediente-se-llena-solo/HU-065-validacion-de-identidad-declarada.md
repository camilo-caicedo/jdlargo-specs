---
id: HU-065
titulo: Validación del nombre e identificación declarados contra los documentos cargados
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-09-17
---

# HU-065 — Validación del nombre e identificación declarados contra los documentos cargados

> **Nota de alcance (2026-09-17).** Esta historia nace de prueba en vivo. `HU-008` ya declaró
> `party.declared_name` y `party.identification` como campos siempre obligatorios al abrir un
> expediente, con origen `declared` (`ADR-0005`) — pero los dejó fijos desde la creación y
> fuera del alcance de conciliación de `HU-019`, que solo compara campos de la matriz de
> requisitos. Esta historia cierra ese hueco: **el nombre y la identificación de la contraparte
> se tratan como cualquier otro dato conciliable, sin que un Administrador tenga que agregarlos
> a la matriz**, porque son estructurales y siempre están presentes.

## Historia

**Como** Analista de Cumplimiento
**quiero** poder corregir el nombre o la identificación que se declararon al crear el
expediente, y que el sistema los contraste automáticamente contra lo que dice el documento de
identificación cargado
**para** detectar de una vez si la contraparte que se declaró es la misma que aparece en sus
propios documentos, sin depender de que alguien lo compare a mano.

## Contexto

Hoy `party.declared_name` y `party.identification` nacen en `HU-008` como afirmaciones de
origen `declared`, y ahí se quedan: no hay forma de editarlas después de crear el expediente, y
la extracción de `HU-017` nunca las recibe como contexto, así que no puede leerlas de un
documento de identidad para compararlas. El resultado es que el dato más básico del
expediente —quién es la contraparte— es el único que nunca pasa por la conciliación que
`HU-019` construyó para todo lo demás.

La solución no es convertirlos en requisitos de la matriz (`HU-007`, `HU-035`): un
Administrador no debería tener que acordarse de configurar "nombre" e "identificación" en cada
tipo de contraparte, porque **siempre** están presentes y **siempre** son obligatorios — es
parte de lo que significa abrir un expediente, no una decisión de configuración. En cambio, se
tratan como un segundo par de campos estructurales, del mismo modo que `dossier.code` o
`dossier.state` no viven en la matriz, pero con la diferencia de que **sí participan en
afirmaciones y conciliación** porque son datos declarados sobre el sujeto, no metadatos del
expediente.

## Criterios de aceptación

```gherkin
Escenario: Corregir el nombre o la identificación declarados
  Dado un expediente abierto con un nombre o identificación declarados
  Cuando el Analista de Cumplimiento corrige ese valor
  Entonces se registra una afirmación nueva de origen `declared`, atribuida a esa persona, con su momento
  Y la afirmación anterior se conserva, no se sobrescribe
  Y el cambio queda registrado en la bitácora
```

```gherkin
Escenario: La extracción recibe el nombre e identificación declarados como contexto
  Dado un expediente con nombre e identificación declarados
  Y un documento de identificación cargado al expediente
  Cuando se ejecuta la extracción sobre ese documento
  Entonces la extracción intenta leer el nombre y la identificación que aparecen en el documento
  Y, si los encuentra, registra una afirmación de origen `extracted` para esos mismos campos
```

```gherkin
Escenario: El nombre o la identificación no coinciden con el documento
  Dado una afirmación declarada y otra extraída del nombre o la identificación, con valores distintos
  Cuando se calcula la conciliación del expediente
  Entonces se abre una discrepancia sobre ese campo, igual que con cualquier campo de la matriz
  Y el expediente no pasa a "pendiente de decisión" mientras esa discrepancia siga abierta
```

```gherkin
Escenario: El nombre y la identificación no requieren configuración de matriz
  Dado un tipo de contraparte cuya matriz no incluye ningún requisito de nombre ni de identificación
  Cuando se abre un expediente de ese tipo
  Entonces el nombre y la identificación declarados igual participan en la conciliación
  Y ningún Administrador necesitó agregarlos a la matriz para que eso ocurra
```

```gherkin
Escenario: La contraparte ve lo que el analista declaró al abrir el expediente
  Dado un expediente con nombre e identificación declarados por el Analista de Cumplimiento al crearlo
  Cuando la contraparte entra con su enlace de acceso
  Entonces ve ese nombre y esa identificación como parte de la información ya registrada del expediente
  Y si más adelante hay una discrepancia sobre esos campos, la ve igual que vería cualquier otra
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la identidad declarada
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta afirmaciones de nombre o identificación con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- `party.declared_name` y `party.identification` son campos **estructurales**: siempre
  obligatorios en todo expediente (`HU-008`), nunca definidos en la matriz de requisitos
  (`HU-007`), pero sí sujetos a afirmación con procedencia y a conciliación (`ADR-0005`,
  `HU-019`), igual que un campo de la matriz.
- Corregir el nombre o la identificación **no edita** la afirmación existente: crea una nueva de
  origen `declared`, atribuida a quien la corrige. La anterior se conserva como evidencia
  (mismo patrón que `HU-020` para las correcciones sobre datos extraídos).
- La extracción (`HU-017`) recibe el nombre y la identificación declarados como parte de su
  contexto de entrada cuando procesa un documento de identificación, para poder producir una
  afirmación `extracted` de esos mismos campos y hacerlos comparables.
- Una discrepancia entre el nombre o la identificación declarados y lo extraído del documento
  sigue exactamente las mismas reglas que cualquier otra discrepancia de `HU-019`: se abre, no
  se cierra sola, y bloquea el paso a "pendiente de decisión" hasta que una persona la resuelva.
- La contraparte ve el nombre y la identificación que el analista declaró al crear el
  expediente, del mismo modo que ve cualquier otro valor vigente del formulario en `HU-022`.

## Fuera de alcance

- La verificación del nombre o la identificación contra una fuente externa (Registraduría,
  Cámara de Comercio) → Fase 3, igual que el resto de verificaciones.
- Que la contraparte pueda editar directamente el nombre o la identificación desde el portal:
  su vía para corregir información sigue siendo la declaración y la firma de `HU-022`.
- Definir "documento de identificación" como un tipo documental nuevo: usa el catálogo que ya
  exista (`HU-013`, `document-type-catalog`).
- Cambiar cómo se calcula qué afirmación es la "vigente" quando hay varias — sigue siendo la
  precedencia configurable de `ADR-0005`, sin cambios.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `assertion.field` | Sí | Incluye ahora `party.declared_name` y `party.identification` como campos conciliables, además de los de la matriz | No |
| `extraction_input.declared_party` | Condicional | Nombre e identificación declarados vigentes; se envía solo si el documento procesado es de tipo identificación | Sí (dato personal) |
| `discrepancy.field` | Sí | Puede ser `party.declared_name` o `party.identification`, además de un campo de matriz | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Historias: extiende `HU-008` (nace el dato), `HU-017` (extracción), `HU-019` (conciliación),
  `HU-020` (corrección atribuida), `HU-022` (lo que ve la contraparte)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna nueva.
- **Depende de:** `HU-005` (afirmaciones), `HU-008`, `HU-013`, `HU-017`, `HU-019`, `HU-020`.
- **Habilita a:** cerrar el hueco más básico de identidad antes de que un expediente llegue a
  decisión — hoy es posible aprobar un expediente sin que nadie haya contrastado si el nombre
  declarado corresponde al del documento de identidad cargado.
- **Riesgo:** si la extracción no logra leer con confianza el nombre de un documento de
  identidad (calidad de imagen, formato no soportado), el campo queda sin afirmación
  `extracted` y no hay nada que conciliar — eso no es un fallo de esta historia, es el mismo
  límite que ya tiene `HU-017` para cualquier campo.
