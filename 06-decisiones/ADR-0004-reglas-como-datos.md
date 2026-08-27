---
id: ADR-0004
titulo: El cumplimiento es configuración versionada, no código
estado: propuesto
fecha: 2026-08-25
---

# ADR-0004 — Reglas y configuración como datos

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
  Oficial de Cumplimiento, no por un desarrollador. Es un producto dentro del producto y se
  subestima siempre.
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

## Preguntas abiertas que dependen de esta decisión

- `PA-017` — ¿Quién configura la matriz de requisitos en la práctica: el cliente por su
  cuenta, o nosotros como servicio de implementación? Cambia por completo el esfuerzo de
  la interfaz de administración.
- `PA-018` — ¿Cuántos estándares y tipos de contraparte hay que soportar en el primer
  cliente? Define si la configuración inicial se carga a mano o necesita interfaz desde el día uno.
