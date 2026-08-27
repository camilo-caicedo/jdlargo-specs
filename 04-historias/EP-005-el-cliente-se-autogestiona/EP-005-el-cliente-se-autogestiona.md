---
id: EP-005
titulo: El cliente se autogestiona
estado: borrador
capacidad: CAP-05
actualizado: 2026-08-27
---

# EP-005 — El cliente se autogestiona

## Objetivo

Poner en manos del cliente la configuración que hasta ahora cargábamos nosotros: estándares,
tipos de contraparte, matriz de requisitos, metodología de riesgo, fuentes, avisos y permisos.
Sin desarrollador de por medio y sin desplegar código.

## Por qué existe

Es la Fase 5 del `02-producto/roadmap.md`, y va tarde **a propósito**. Hasta aquí el motor
configurable existe desde `EP-000`; lo que no existe es la interfaz para manejarlo. Mientras haya
un solo cliente ancla, configurarlo a mano es más rápido que construir la pantalla.

Y la frase del roadmap que conviene no perder de vista:

> La Fase 5 es, en realidad, **la fase que convierte esto en un producto vendible a un segundo
> cliente**.

`ADR-0004` ya advirtió que esta interfaz es *"un producto dentro del producto y se subestima
siempre"*, y añade el requisito difícil: tiene que ser usable por un **Oficial de Cumplimiento**,
no por un desarrollador. Alguien que entiende de riesgo LA/FT y no de esquemas de datos.

## Alcance

**Incluye:**

- Administración de versiones de configuración: crear borrador, comparar, publicar (`HU-034`).
- Tipos de contraparte y matriz de requisitos (`HU-035`).
- Metodología de riesgo (`HU-036`).
- Catálogo de fuentes y aviso de privacidad (`HU-037`).
- Usuarios, roles y permisos (`HU-038`).
- Prueba de la configuración contra un expediente de ensayo antes de publicar (`HU-039`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Cambios en el motor de reglas | — | Aquí se administra la configuración, no se amplía el conjunto cerrado de `ADR-0004` |
| Panel de indicadores del Oficial de Cumplimiento | Fase 6 | Es una vista de operación, no de configuración |
| Administración de planes y cobro | Fase C | — |
| Editor libre de reglas o lenguaje de expresiones | Sin fase | `ADR-0004` lo descarta expresamente |
| Migración masiva de configuración entre organizaciones clientes | Sin fase | §31: la configuración no se comparte entre clientes del SaaS |
| Portal de administración para nosotros como proveedores | Sin definir | No hay definición del cliente |

## Actores involucrados

- **Administrador** — configura la mayor parte: tipos, matriz, fuentes, usuarios.
- **Oficial de Cumplimiento** — es quien publica: la metodología y la matriz son de su
  responsabilidad legal.
- **Auditor / Consulta** — ve la configuración y su historial sin poder tocarlos.

## Criterios de éxito

1. **Cero intervenciones nuestras** para que un cliente cambie su matriz, su metodología o sus
   fuentes.
2. **Cero despliegues** asociados a un cambio de configuración.
3. Un Oficial de Cumplimiento sin conocimientos técnicos completa un cambio de principio a fin y
   entiende qué va a pasar antes de publicar.
4. **Ninguna publicación a ciegas**: siempre se puede comparar contra la versión vigente y probar
   antes de publicar.
5. Ninguna operación de esta épica permite editar una versión ya publicada.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-034` | Administrar versiones de configuración | Must | borrador |
| `HU-035` | Administrar tipos de contraparte y matriz de requisitos | Must | borrador |
| `HU-036` | Administrar la metodología de riesgo | Must | borrador |
| `HU-037` | Administrar fuentes y aviso de privacidad | Should | borrador |
| `HU-038` | Administrar usuarios, roles y permisos | Must | borrador |
| `HU-039` | Probar la configuración antes de publicar | Should | borrador |

Orden sugerido: `HU-034` → `HU-035` → `HU-039` → `HU-036` → `HU-038` → `HU-037`. La prueba se
adelanta porque sin ella la matriz se publica a ciegas, y una matriz mal publicada afecta a todos
los expedientes que se abran después.

## Dependencias

- **Épicas:** `EP-000` a `EP-004`. Esta épica **no añade capacidades nuevas al motor**: le pone
  interfaz a lo que ya existe.
- **Preguntas abiertas:** `PA-017` — la que decide el tamaño real de esta épica: si el cliente
  configura por su cuenta, la interfaz es el producto; si lo hacemos nosotros como servicio de
  implementación, buena parte de esta épica se puede recortar. `PA-018`, `PA-024`.
- **Decisiones:** `ADR-0004`.

## Riesgo abierto

**`PA-017` puede reducir esta épica a la mitad o duplicarla.** Es la pregunta de mayor impacto
sobre el esfuerzo de todo el proyecto y sigue abierta. Conviene responderla antes de estimar la
Fase 5, no durante.

El segundo riesgo es de diseño, no de alcance: una interfaz que exponga el modelo de datos tal
cual —tablas, claves, condiciones anidadas— cumple el criterio de existir y falla el de ser
usable. La prueba honesta es la del criterio de éxito 3, y conviene hacerla con el Oficial de
Cumplimiento real antes de dar la fase por terminada.
