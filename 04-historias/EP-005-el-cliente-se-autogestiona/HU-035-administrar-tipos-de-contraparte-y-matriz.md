---
id: HU-035
titulo: Administrar tipos de contraparte y matriz de requisitos
estado: borrador
epica: EP-005
prioridad: Must
actualizado: 2026-08-27
---

# HU-035 — Administrar tipos de contraparte y matriz de requisitos

## Historia

**Como** Administrador de una organización cliente
**quiero** definir yo mismo qué campos y qué documentos se le exigen a cada tipo de contraparte
**para** ajustar lo que pedimos cuando cambia una norma o cuando aparece un tipo de proveedor
nuevo, sin esperar a que alguien lo programe.

## Contexto

La matriz de requisitos es, en palabras del documento del cliente, **el módulo más importante**.
`HU-007` la construyó como datos y la cargamos a mano; esta historia la pone en manos del
cliente.

Aquí es donde `ADR-0004` cobra su promesa: *"cuando cambie una norma, lo que se actualiza es la
configuración y su versión, no hay que reconstruir el software completo"*. Si esta pantalla
funciona, esa frase es cierta. Si no, es una intención.

El reto de diseño es que la matriz es multidimensional —estándar × tipo de contraparte ×
requisito, con condicionales— y quien la maneja piensa en cumplimiento, no en tablas cruzadas.

## Criterios de aceptación

```gherkin
Escenario: Crear un tipo de contraparte
  Dado un borrador de configuración
  Cuando el Administrador crea el tipo de contraparte "transportadora aliada" indicando su naturaleza
  Entonces el tipo queda disponible en el borrador
  Y todavía no afecta a ningún expediente, porque el borrador no está publicado
```

```gherkin
Escenario: Definir qué exige una combinación
  Dado un borrador con un estándar y un tipo de contraparte
  Cuando el Administrador marca qué campos y qué tipos documentales son obligatorios para esa combinación
  Entonces la matriz refleja esos requisitos
  Y se puede ver la combinación completa en una sola vista
```

```gherkin
Escenario: Definir un requisito condicional sin escribir código
  Dado un borrador de matriz
  Cuando el Administrador define que cierto documento se exige solo si la contraparte declara ser PEP
  Entonces la condición queda registrada usando el conjunto cerrado de condiciones disponibles
  Y el Administrador la elige de una lista, sin escribir ninguna expresión
```

```gherkin
Escenario: No se puede escribir una condición libre
  Dado un intento de definir una condición fuera del conjunto disponible
  Cuando se guarda el requisito
  Entonces la operación es rechazada
  Y se indica qué condiciones admite el sistema
```

```gherkin
Escenario: Ver el impacto antes de publicar
  Dado un borrador que añade un documento obligatorio a un tipo de contraparte
  Cuando el Administrador pide ver el impacto
  Entonces se le indica a qué tipo afecta y desde cuándo aplicaría
  Y se le indica cuántos expedientes abiertos usan la versión anterior
```

```gherkin
Escenario: Una combinación sin requisitos bloquea la publicación
  Dado un borrador con un tipo de contraparte sin ningún requisito definido
  Cuando se intenta publicar la versión
  Entonces la publicación es rechazada indicando qué combinación quedó vacía
```

```gherkin
Escenario: Retirar un tipo de contraparte no borra su historia
  Dado un tipo de contraparte con expedientes ya abiertos
  Cuando el Administrador lo retira en un borrador y publica
  Entonces no se pueden abrir expedientes nuevos de ese tipo
  Y los expedientes existentes conservan su tipo y su matriz de la versión con la que se abrieron
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la matriz
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando administra la matriz con su contexto de usuario propagado
  Entonces solo ve y modifica la de "Alfa Ficticia S.A.S."
  Y no puede ver la de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- Los cambios se hacen **siempre sobre un borrador** (`HU-034`). Nunca sobre una versión
  publicada.
- Las condiciones se **eligen** de un conjunto cerrado y versionado; el Administrador no escribe
  expresiones (`ADR-0004`).
- Antes de publicar, el sistema muestra el impacto: a qué tipos afecta, desde cuándo, y cuántos
  expedientes siguen en la versión anterior.
- Retirar un tipo de contraparte impide abrir expedientes nuevos, pero **no altera los
  existentes**, que citan su propia versión.
- Una combinación de estándar y tipo sin requisitos no se publica.
- La plantilla propia del sector transporte se resuelve creando su estándar y sus tipos, no
  pidiendo una excepción en el código (Fase 6 del documento del cliente).
- Administrar la matriz requiere permiso explícito; publicarla, el de publicar configuración.

## Fuera de alcance

- El mecanismo de versiones → `HU-034`.
- La prueba de la matriz contra un expediente de ensayo → `HU-039`.
- La metodología de riesgo → `HU-036`.
- La ampliación del conjunto de condiciones disponibles: es un cambio del producto, no del
  cliente.
- La importación de matrices desde una hoja de cálculo: útil, pero no definido por el cliente.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `tipo_contraparte.nombre` | Sí | Único dentro de la versión | No |
| `tipo_contraparte.naturaleza` | Sí | `persona_natural` \| `persona_juridica` | No |
| `tipo_contraparte.estado` | Sí | `activo` \| `retirado` | No |
| `requisito.clase` | Sí | `campo` \| `tipo_documental` | No |
| `requisito.obligatorio` | Sí | `siempre` \| `condicional` \| `opcional` | No |
| `requisito.condicion` | Condicional | Solo del conjunto cerrado; obligatoria si es condicional | No |
| Cobertura de la matriz | Sí | Toda combinación activa tiene al menos un requisito | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: Fase 1, Fase 2, Fase 6, §41
- Decisiones: `ADR-0004`
- Historias: pone interfaz a `HU-007`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-017` (quién configura), `PA-018` (cuántos estándares y tipos).
  **Queda en `borrador`.**
- **Supuestos:** `SUP-001`, `SUP-008` (la norma del sector transporte no está verificada: se
  soporta como configuración, no como regla afirmada).
- **Depende de:** `HU-007`, `HU-034`, `HU-003`.
- **Habilita a:** `HU-039`, y que un segundo cliente se configure sin nosotros.
- **Riesgo:** es la pantalla más difícil del producto. Una matriz multidimensional con
  condicionales presentada como una tabla cruzada es ilegible, y presentada como un formulario
  por requisito es interminable. Conviene prototiparla con el Oficial de Cumplimiento real antes
  de construirla.
