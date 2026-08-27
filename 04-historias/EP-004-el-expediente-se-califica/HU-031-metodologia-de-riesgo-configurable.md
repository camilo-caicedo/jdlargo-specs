---
id: HU-031
titulo: Metodología de riesgo configurable
estado: borrador
epica: EP-004
prioridad: Must
actualizado: 2026-08-27
---

# HU-031 — Metodología de riesgo configurable

## Historia

**Como** Oficial de Cumplimiento
**quiero** definir mis propios factores de riesgo, sus pesos, sus escalas y sus umbrales
**para** que la plataforma aplique **mi** metodología, que es de la que respondo, y no una que
venga impuesta por el proveedor del software.

## Contexto

Es la Fase 14 y cierra `PA-006`: **no hay una metodología única**. Cada cliente configura la
suya, y eso no es una concesión sino el reparto de responsabilidad que fija la §40 —el cliente
responde por su metodología y por cómo clasifica el riesgo; el proveedor, por que la plataforma
la aplique bien.

Los factores que la §14 enumera: tipo de contraparte, producto o servicio, canal, jurisdicción,
actividad económica, condición de PEP, resultados de listas, antecedentes, volumen de operación,
estructura societaria, comportamiento y señales de alerta. Con sus ponderaciones, escalas,
umbrales y reglas de escalamiento, **con fecha de vigencia y versión**.

`ADR-0004` añade la restricción de forma: el conjunto de condiciones y acciones es cerrado y
versionado, evaluado por código propio. Nada de un lenguaje libre dentro de la configuración: en
un sistema multi-tenant eso es una superficie de ejecución arbitraria, imposible de auditar e
indepurable para quien la escribe.

## Criterios de aceptación

```gherkin
Escenario: Configurar una metodología y publicarla
  Dado una versión de configuración en borrador
  Cuando se definen los factores de riesgo, sus ponderaciones, la escala de niveles y los umbrales
  Y se publica la versión
  Entonces la metodología queda vigente para las evaluaciones posteriores
  Y ningún factor, peso ni umbral aparece escrito en el código del programa
```

```gherkin
Escenario: Dos organizaciones clientes, dos metodologías
  Dado dos organizaciones clientes con metodologías distintas
  Cuando se evalúa un expediente equivalente en cada una
  Entonces cada una obtiene el nivel que su metodología produce
  Y la diferencia proviene de la configuración, no del programa
```

```gherkin
Escenario: Una metodología publicada no se edita
  Dado una metodología publicada con expedientes ya evaluados
  Cuando se intenta modificar uno de sus factores o umbrales
  Entonces la operación es rechazada
  Y la única vía es publicar una versión nueva
```

```gherkin
Escenario: Reglas de escalamiento
  Dado una metodología que define que cierta combinación de factores exige debida diligencia intensificada
  Cuando un expediente cumple esa combinación
  Entonces la evaluación produce la acción de escalamiento correspondiente
  Y la acción proviene del conjunto cerrado de acciones, no de código escrito para ese cliente
```

```gherkin
Escenario: Solo se admiten condiciones del conjunto cerrado
  Dado un intento de configurar una regla con una expresión libre o un fragmento de código
  Cuando se valida la configuración
  Entonces la publicación es rechazada
  Y se indica qué tipos de condición admite el motor
```

```gherkin
Escenario: Una metodología incoherente no se publica
  Dado una metodología cuyos umbrales dejan un rango de puntaje sin nivel asignado
  Cuando se intenta publicar
  Entonces la publicación es rechazada indicando el rango sin cubrir
  Y la versión permanece en borrador
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la metodología
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta metodologías con su contexto de usuario propagado
  Entonces obtiene únicamente las de "Alfa Ficticia S.A.S."
  Y un intento de publicar en "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- La metodología es **contenido de una versión de configuración** (`HU-004`): inmutable una vez
  publicada, con fecha de vigencia y número de versión (§14, §41).
- Se configuran: factores, ponderaciones, escalas, umbrales y reglas de escalamiento. Ninguno de
  esos valores aparece en el código.
- Las condiciones admitidas son las del conjunto cerrado de `ADR-0004`: campo, operador, valor y
  combinadores. Las acciones también: exigir campo, exigir documento, exigir fuente, sumar factor
  de riesgo, disparar diligencia intensificada, crear alerta, fijar periodicidad.
- **No se admite un lenguaje libre ni expresiones arbitrarias** dentro de la configuración.
- Una metodología no se puede publicar si es incoherente: rangos sin nivel, pesos que no suman lo
  que declara la escala, o factores referidos que no existen.
- Publicar una metodología requiere permiso explícito. Según la matriz base, modificar la
  metodología es competencia del Oficial de Cumplimiento (§30).
- La metodología puede definir factores distintos por tipo de contraparte y por estándar: es
  configuración, no una tabla universal.

## Fuera de alcance

- El cálculo en sí y su explicación → `HU-032`.
- La interfaz para editar la metodología → Fase 5. Aquí se carga por operación de sistema.
- La periodicidad de actualización que la metodología también alimenta → Fase 6.
- Modelos de aprendizaje automático o puntajes dinámicos por comportamiento → fase posterior
  (§36).
- La validación de si la metodología del cliente es correcta desde el punto de vista regulatorio:
  es responsabilidad del cliente (§40).

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `metodologia.organization_id` | Sí | Organización cliente existente | No |
| `metodologia.version_configuracion_id` | Sí | Versión existente (`HU-004`) | No |
| `metodologia.fecha_vigencia_desde` | Sí | Fecha; no anterior a la de la versión previa | No |
| `factor.clave` | Sí | Factor del catálogo de la §14 o definido por el cliente | No |
| `factor.ponderacion` | Sí | Numérica; coherente con la escala declarada | No |
| `escala.niveles` | Sí | Al menos dos niveles, sin rangos superpuestos ni huecos | No |
| `umbral.desde` / `hasta` | Sí | Cubren toda la escala sin dejar rangos sin nivel | No |
| `regla.condicion` | Sí | Solo condiciones del conjunto cerrado | No |
| `regla.accion` | Sí | Solo acciones del conjunto cerrado | No |
| `metodologia.estado` | Sí | `borrador` \| `publicada` \| `reemplazada` | No |

## Trazabilidad

- Épica: `EP-004`
- Capacidad: `CAP-04`
- Documento del cliente: Fase 14, §34, §40, §41, §44 pregunta I
- Decisiones: `ADR-0004` (conjunto cerrado de condiciones y acciones; sin lenguaje embebido)
- Preguntas: cierra la parte configurable de `PA-006`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-017` (quién configura en la práctica) y `PA-018` (cuántos
  estándares y tipos), que determinan si esto es viable sin la interfaz de la Fase 5. **Queda en
  `borrador`.**
- **Supuestos:** `SUP-001`.
- **Depende de:** `HU-004` (versiones), `HU-003` (permiso de modificar la metodología).
- **Habilita a:** `HU-032`, `HU-033`, y la periodicidad de la Fase 6.
- **Riesgo:** la presión por "solo esta regla en código, que es rara" aparece siempre y es la que
  bifurca el producto por cliente. Si una regla no cabe en el conjunto cerrado, la respuesta
  correcta es ampliar el conjunto —para todos y versionado—, no escribirla a mano para uno.
