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
> expediente, con origen `declared` (`ADR-0005`) — pero los dejó fuera del alcance de
> conciliación de `HU-019`, que solo compara campos de la matriz de requisitos. Esta historia
> cierra ese hueco: **el nombre y la identificación de la contraparte se tratan como cualquier
> otro dato conciliable, sin que un Administrador tenga que agregarlos a la matriz**, porque
> son estructurales y siempre están presentes.
>
> **Corrección de diseño (2026-09-17, antes de la primera implementación).** Un primer borrador
> de esta historia proponía que el Analista de Cumplimiento corrigiera directamente el nombre o
> la identificación declarados. Eso contradice un límite ya establecido y deliberado en
> `HU-061`: *"Editar los datos declarados por la contraparte (nombre, identificación) ... se
> corrige por el flujo de HU-014 (solicitud de correcciones), no reescribiéndola directo"* —
> precisamente para que quien declaró un dato sea quien lo corrige, y quede su firma sobre el
> cambio, no la de quien lo detectó. Esta historia se reescribió para **reutilizar
> `HU-014`/`HU-022` tal como existen**, no para abrir una vía de edición nueva y paralela.

## Historia

**Como** Analista de Cumplimiento
**quiero** que el nombre y la identificación declarados al crear el expediente se contrasten
automáticamente contra lo que dice el documento de identificación cargado, y poder pedirle a la
contraparte que lo confirme o lo corrija cuando no coinciden
**para** detectar si la contraparte que se declaró es la misma que aparece en sus propios
documentos, sin depender de que alguien lo compare a mano ni de reescribir yo mismo un dato que
no declaré.

## Contexto

Hoy `party.declared_name` y `party.identification` nacen en `HU-008` como afirmaciones de
origen `declared`, y la extracción de `HU-017` nunca las recibe como contexto, así que no puede
leerlas de un documento de identidad para compararlas. El resultado es que el dato más básico
del expediente —quién es la contraparte— es el único que nunca pasa por la conciliación que
`HU-019` construyó para todo lo demás, ni por el ciclo de corrección de `HU-014` que ya existe
para cualquier otro dato declarado.

La solución no es convertirlos en requisitos de la matriz (`HU-007`, `HU-035`): un
Administrador no debería tener que acordarse de configurar "nombre" e "identificación" en cada
tipo de contraparte, porque **siempre** están presentes y **siempre** son obligatorios — es
parte de lo que significa abrir un expediente, no una decisión de configuración. Tampoco es
abrir una vía de edición directa para el analista: `HU-061` ya decidió que un dato declarado se
corrige pidiéndoselo a quien lo declaró, y esta historia sigue esa misma regla en vez de
excepcionarla.

## Criterios de aceptación

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
Escenario: Pedir a la contraparte que confirme o corrija su nombre o identificación
  Dado una discrepancia abierta sobre el nombre o la identificación declarados
  Cuando el Analista de Cumplimiento solicita correcciones (`HU-014`)
  Entonces la observación indica puntualmente que el nombre o la identificación no coinciden con el documento cargado
  Y la contraparte, no el analista, es quien registra la afirmación declarada nueva al corregir
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
  Y si hay una discrepancia sobre esos campos, la ve igual que vería cualquier otra, con su origen correcto (`HU-022`)
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
- **Nadie más que quien declaró el dato registra su corrección.** El nombre y la identificación
  declarados siguen la misma regla que cualquier otro dato de origen `declared`: se corrigen
  por el flujo de solicitud de correcciones (`HU-014`), nunca por edición directa de un
  analista (`HU-061`).
- La extracción (`HU-017`) recibe el nombre y la identificación declarados como parte de su
  contexto de entrada cuando procesa un documento de identificación, para poder producir una
  afirmación `extracted` de esos mismos campos y hacerlos comparables.
- Una discrepancia entre el nombre o la identificación declarados y lo extraído del documento
  sigue exactamente las mismas reglas que cualquier otra discrepancia de `HU-019`: se abre, no
  se cierra sola, y bloquea el paso a "pendiente de decisión" hasta que se resuelva.
- La contraparte ve el nombre y la identificación que el analista declaró al crear el
  expediente, del mismo modo que ve cualquier otro valor vigente del formulario en `HU-022`, y
  con el origen correctamente etiquetado si hubo una corrección de por medio.

## Fuera de alcance

- La edición directa del nombre o la identificación por parte de un analista u oficial de
  cumplimiento — sigue explícitamente vetada por `HU-061`.
- La verificación del nombre o la identificación contra una fuente externa (Registraduría,
  Cámara de Comercio) → Fase 3, igual que el resto de verificaciones.
- Definir "documento de identificación" como un tipo documental nuevo: usa el catálogo que ya
  exista (`HU-013`, `document-type-catalog`).
- Cambiar cómo se calcula qué afirmación es la "vigente" cuando hay varias — sigue siendo la
  precedencia configurable de `ADR-0005`, sin cambios.
- Cualquier cambio al mecanismo de "solicitar correcciones" en sí (`HU-014`): esta historia lo
  reutiliza tal como existe, no lo modifica.

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
  `HU-014` (vía de corrección, sin cambios), `HU-022` (lo que ve la contraparte); respeta el
  límite de `HU-061` (ninguna edición directa de datos declarados)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna nueva.
- **Depende de:** `HU-005` (afirmaciones), `HU-008`, `HU-013`, `HU-014`, `HU-017`, `HU-019`.
- **Habilita a:** cerrar el hueco más básico de identidad antes de que un expediente llegue a
  decisión — hoy es posible aprobar un expediente sin que nadie haya contrastado si el nombre
  declarado corresponde al del documento de identidad cargado.
- **Riesgo:** si la extracción no logra leer con confianza el nombre de un documento de
  identidad (calidad de imagen, formato no soportado), el campo queda sin afirmación
  `extracted` y no hay nada que conciliar — eso no es un fallo de esta historia, es el mismo
  límite que ya tiene `HU-017` para cualquier campo.
