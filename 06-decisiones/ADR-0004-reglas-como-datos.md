---
id: ADR-0004
titulo: El cumplimiento es configuración versionada, no código
estado: propuesto
fecha: 2026-08-25
actualizado: 2026-09-05
---

# ADR-0004 — Reglas y configuración como datos

> **Actualización 2026-09-05.** Las respuestas a `PA-017`, `PA-018` y `PA-025` cierran las
> tres incógnitas que este ADR dejaba abiertas: quién configura, cuánto hay que configurar en
> el primer cliente, y cuándo congela un expediente su versión. Se añaden las secciones 2b y 6.

## Contexto

El [documento del cliente](../01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md)
impone una restricción que no estaba en `ADR-0001` y que cambia la naturaleza del sistema:

> **"No se debe programar SAGRILAFT como una estructura fija en el código"** (Fase 0)
>
> **"Cuando cambie una norma, lo que se actualiza es la configuración/versión de las
> reglas — no hay que reconstruir el software completo"** (§41)

Cuatro piezas del producto son configurables **por cada cliente del SaaS**:

| Pieza | Documento | Qué configura el cliente |
|---|---|---|
| Matriz de requisitos | Fase 2 | Qué campo, documento y fuente aplica a cada combinación de estándar × tipo de contraparte |
| Motor de riesgo | Fase 14 | Factores, ponderaciones, escalas, umbrales y reglas de escalamiento |
| Catálogo de fuentes | Fase 11 | Qué fuentes externas se consultan y para qué |
| Motor de relaciones | Fase 10 | Qué tratamiento requiere cada tipo de persona relacionada |
| RBAC | §30 | Qué puede hacer cada rol, "según la política del cliente" |

Esto **no es una aplicación con reglas de negocio**. Es un motor que ejecuta reglas que
otros escriben. La diferencia es la misma que hay entre una calculadora de impuestos y una
hoja de cálculo.

## Decisión

### 1. La configuración de cumplimiento vive en la base de datos, no en el código

Ninguna referencia a "SAGRILAFT", "SARLAFT", un umbral de riesgo, un documento obligatorio
o un factor de ponderación aparece literalmente en el código de la aplicación. Todo eso son
filas.

El código conoce la **forma** de una regla; nunca su **contenido**.

### 2. Todo lo configurable es inmutable y versionado

Una matriz de requisitos, una metodología de riesgo o un formulario **no se editan**: se
publica una versión nueva. La anterior queda intacta porque hay expedientes evaluados con
ella.

Cada expediente guarda, congelado en el momento de su evaluación: estándar, versión
normativa, versión de la metodología, versión de la matriz, versión de las reglas y versión
del formulario (§41). Sin esto, una auditoría a dos años no puede reconstruir por qué el
sistema pidió lo que pidió.

> Consecuencia práctica: **no existe `UPDATE` sobre una tabla de configuración publicada.**
> Solo `INSERT` de una versión nueva y cambio del puntero de "vigente".

### 2b. Cuándo se congela la versión de un expediente

Cierra `PA-025`. El cliente lo respondió con un ejemplo: un formato de vinculación congela su
versión **al abrirse**, y cuando se vuelva a hacer debida diligencia sobre esa contraparte, se
hará bajo la versión más reciente. Lo mismo aplica a la matriz de requisitos.

La regla completa, en tres partes:

| Momento | Qué se congela | Por qué |
|---|---|---|
| **Al abrir el expediente** | Formulario, matriz de requisitos y tipos documentales | La contraparte no puede ver cambiar lo que se le pide mientras lo está diligenciando |
| **Al evaluar** | Metodología de riesgo, reglas, versión normativa y catálogo de fuentes | La evaluación tiene que ser reproducible con exactamente esas reglas |
| **Al cerrar** | Todo lo anterior, de forma definitiva | **Una evaluación cerrada es inmutable.** Publicar una versión nueva jamás reescribe hacia atrás un expediente ya decidido |

Los expedientes **en curso** son el caso intermedio: siguen con la versión que congelaron al
abrirse. Si entra en vigencia una versión nueva, el sistema puede **marcarlos para
revalidación** —no migrarlos en silencio—, y la migración solo ocurre si el cliente la
autoriza de forma explícita, con su propio registro en bitácora.

> Corolario: la pregunta "¿con qué reglas se evaluó este expediente?" tiene que poder
> responderse sin conjeturas. Es la razón entera de este ADR.

### 3. Los formularios se generan desde la matriz, no se escriben a mano

La Fase 6 exige formularios distintos por tipo de contraparte, definidos en la Fase 2. Eso
significa **formularios dinámicos**: campos, validaciones, obligatoriedad y condicionalidad
salen de la configuración.

Consecuencia técnica concreta que contradice a `ADR-0001`: los esquemas Zod del expediente
**no pueden ser estáticos**. Hace falta un constructor que arme el esquema en tiempo de
ejecución desde la definición almacenada. Zod estático sigue siendo correcto para todo lo
demás (autenticación, facturación, administración) — pero no para el formulario de
diligenciamiento.

### 4. El motor de reglas es un evaluador propio y acotado

**No** se adopta un motor de reglas de propósito general ni un lenguaje embebido. Se define
un conjunto cerrado y versionado de tipos de condición y de acción, evaluado por código
propio:

```
Condición: campo | operador | valor | (combinadores y/o)
Acción:    exigir campo · exigir documento · exigir fuente · sumar factor de riesgo
           · disparar DDI · crear alerta · fijar periodicidad
```

Razones para no usar algo genérico: un lenguaje libre dentro de la configuración es una
superficie de ejecución arbitraria en un sistema multi-tenant, es imposible de auditar, y
resulta indepurable para el cliente que lo escribe. Un conjunto cerrado se puede validar,
explicar y probar.

### 5. Toda evaluación deja rastro explicable

Evaluar nunca devuelve solo un resultado: devuelve **qué reglas se dispararon, con qué
datos de entrada y con qué versión**. Es lo que sostiene el criterio de "cumplimiento
verificable" (§42) y las 16 preguntas del criterio de aceptación (§44).

### 6. La configuración se entrega con plantillas base, y la interfaz entra al MVP

Cierra `PA-017` y `PA-018`. El modelo es **híbrido**: nosotros entregamos plantillas
preconfiguradas por régimen y sector, y el cliente las ajusta a su política interna. Ni el
cliente construye desde cero, ni el SaaS le impone una política única.

Lo que eso implica, y que cambia el alcance respecto de la versión anterior de este ADR:

- **La interfaz de administración de la configuración entra al MVP**, no a una fase tardía.
  Es lo que el cliente pidió expresamente. Lo que sí puede fasearse es su profundidad.
- **La configuración inicial se presta como servicio de implementación** para el primer
  cliente. Es un servicio vendible, no un costo oculto.
- Cuatro piezas de interfaz mínimas: **asistente de configuración**, **duplicar plantilla**,
  **versionar** y un **simulador** que responda "¿qué le voy a pedir a esta contraparte?"
  antes de publicar (`HU-039`).

**Volumen del primer despliegue** (`PA-018`): dos estándares —**SARLAFT y PTEE**— y un
conjunto controlado de tipos de contraparte, configurados desde el inicio según su tipo:
conductor, propietario, poseedor, proveedor, cliente, empleado, accionista. No es una lista
cerrada: es el arranque. El motor sigue siendo genérico y admite persona natural, persona
jurídica y los tipos que cada sector traiga.

> El error a evitar es el de siempre: modelar exactamente esos dos estándares y esos siete
> tipos. El primer despliegue es un dato de carga, no una especificación del motor.

## Consecuencias

**A favor**

- La norma cambia y el producto se actualiza publicando una versión de configuración, sin
  desplegar código. Era el requisito explícito del cliente.
- Cada cliente del SaaS tiene su propia metodología sin bifurcar el producto.
- El versionamiento es simultáneamente requisito de auditoría y mecanismo de migración.

**Costo que se asume**

- **El proyecto es entre dos y tres veces más grande de lo que suponía `ADR-0001`.** Un
  motor configurable con formularios dinámicos y versionamiento no es una aplicación CRUD
  con más pantallas: es otra categoría de sistema. Ver el replanteo en `02-producto/roadmap.md`.
- Hace falta una **interfaz de administración de la configuración** que sea usable por un
  Oficial de Cumplimiento, no por un desarrollador. Es un producto dentro del producto, se
  subestima siempre, y ahora además **está dentro del MVP** (`PA-017`).
- El versionamiento inmutable complica las consultas: casi todo lo de configuración lleva
  una dimensión temporal.
- Depurar "por qué el sistema pidió este documento" exige que la explicabilidad de la
  regla 5 exista desde el primer día. Añadirla después es reescribir el evaluador.

## Alternativas descartadas

| Alternativa | Por qué no |
|---|---|
| Reglas en código con banderas por cliente | Es exactamente lo que el cliente prohíbe. Cada cambio normativo sería un despliegue, y cada cliente nuevo una bifurcación. |
| Motor de reglas de propósito general (JSON Logic, CEL, un lenguaje embebido) | Superficie de ejecución arbitraria en multi-tenant, imposible de auditar y de explicar al usuario que la configura. |
| Configuración editable en sitio, sin versionar | Rompe la auditoría: un expediente de hace un año ya no se puede reconstruir. Es el fallo que el documento señala expresamente. |
| Un esquema de base de datos por cliente | Aislamiento por conveniencia, pesadilla de migración. `ADR-0001` ya optó por RLS. |

## Preguntas abiertas que dependían de esta decisión

Las tres están **resueltas** (2026-09-05):

- `PA-017` — configura el cliente, sobre plantillas base que entregamos nosotros. La interfaz
  de administración entra al MVP y la configuración inicial se presta como servicio → §6.
- `PA-018` — dos estándares (SARLAFT y PTEE) y siete tipos de contraparte en el primer
  despliegue, como carga inicial administrada → §6.
- `PA-025` — el expediente congela formulario y matriz al abrirse, y metodología y reglas al
  evaluar; lo cerrado es inmutable → §2b.
