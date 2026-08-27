---
id: HU-009
titulo: Máquina de estados del expediente
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-009 — Máquina de estados del expediente

## Historia

**Como** Sistema
**quiero** que el avance de un expediente solo pueda ocurrir por transiciones declaradas como
datos, y que cada una quede registrada con quién la provocó y por qué
**para** que el estado de un expediente sea siempre explicable y no dependa de que cada parte
del programa actualice bien una columna de texto.

## Contexto

La §38 define la ruta principal del expediente y una ruta alternativa cuando hay alerta o
riesgo alto. `08-desarrollo/arquitectura-de-aplicacion.md` decide cómo se implementa: **como
máquina de estados en la base de datos**, no como una columna que cada parte del código
actualiza a su antojo. Las transiciones válidas son datos, y una transición inválida es un
error de dominio.

La consecuencia útil: **la bitácora del expediente es la lista de sus transiciones**. La
pregunta "¿por qué este expediente lleva tres semanas parado?" se contesta mirando la última
transición y quién la hizo.

En la Fase 1 solo existe la ruta principal recortada: sin verificación externa, sin screening y
sin análisis de riesgo, que llegan en las fases 3 y 4. Los estados de esas fases se declaran
igualmente, pero ningún camino de la Fase 1 los alcanza.

## Criterios de aceptación

```gherkin
Escenario: Una transición válida avanza el expediente y deja registro
  Dado un expediente en el estado "en diligenciamiento"
  Y una transición declarada como válida desde ese estado hacia "documentos recibidos"
  Cuando se ejecuta esa transición
  Entonces el expediente queda en "documentos recibidos"
  Y queda registrada la transición con quién la provocó, cuándo y con qué motivo
  Y la transición aparece en la bitácora del expediente
```

```gherkin
Escenario: Una transición no declarada es rechazada
  Dado un expediente en el estado "enviada"
  Y que no existe transición declarada desde "enviada" hacia "aprobada"
  Cuando se intenta llevar el expediente directamente a "aprobada"
  Entonces la operación es rechazada como error de dominio
  Y el expediente permanece en "enviada"
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: El estado no se puede cambiar por fuera de la máquina
  Dado un expediente en cualquier estado
  Cuando se intenta modificar directamente su columna de estado sin ejecutar una transición
  Entonces la operación es rechazada por la base de datos
  Y no queda registro de cambio de estado
```

```gherkin
Escenario: La historia del expediente es la lista de sus transiciones
  Dado un expediente que ha recorrido varios estados
  Cuando se consulta su historia
  Entonces se obtiene la secuencia ordenada de todas sus transiciones
  Y cada una indica el estado de origen, el de destino, quién la provocó, cuándo y por qué
  Y la secuencia no tiene huecos desde la apertura del expediente
```

```gherkin
Escenario: Cada transición exige el permiso correspondiente
  Dado un usuario cuyo rol vigente no incluye el permiso asociado a una transición
  Cuando intenta ejecutarla
  Entonces la transición es rechazada
  Y el expediente no cambia de estado
```

```gherkin
Escenario: Un estado final no admite salida
  Dado un expediente en un estado final de la Fase 1
  Cuando se intenta ejecutar cualquier transición desde él
  Entonces la operación es rechazada
  Y el expediente conserva su estado y su decisión intactos
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre estados y transiciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta expedientes y sus transiciones con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y un intento de ejecutar una transición sobre un expediente de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Los estados y las transiciones válidas son **datos**, no un `switch` en el programa. Añadir un
  estado no debería exigir reescribir la lógica de avance.
- Una transición se persiste con: expediente, estado de origen, estado de destino, quién la
  provocó, tipo de actor, momento, motivo y versión de configuración vigente.
- **El estado del expediente solo cambia ejecutando una transición.** No existe escritura
  directa sobre el estado, ni siquiera desde procesos de sistema.
- Cada transición declara qué permiso exige (`HU-003`). Que un estado sea alcanzable no
  significa que cualquiera pueda alcanzarlo.
- La ruta principal de la Fase 1 es la de la §38, recortada a lo que existe:
  `borrador → enviada → en diligenciamiento → documentos recibidos → en revisión → pendiente de
  decisión → aprobada | aprobada con condiciones | rechazada → cerrada`.
- Los estados de verificación, screening, análisis de riesgo y debida diligencia intensificada
  se declaran desde ahora, pero **ninguna transición de la Fase 1 los alcanza**. Se conectan en
  las fases 3 y 4 añadiendo transiciones, no reescribiendo la máquina.
- Un expediente cerrado es final: no vuelve atrás. Reabrirlo, si el cliente lo necesitara, sería
  una transición declarada y registrada, nunca una edición.

## Fuera de alcance

- La ruta alternativa por alerta o riesgo alto (§38) → fases 3 y 4. Se añaden transiciones, no
  se rehace nada.
- Los estados del documento, que son su propia máquina → `HU-013` para el mínimo de la Fase 1, y
  Fase 2 para las vigencias.
- Plazos automáticos y transiciones disparadas por el paso del tiempo → Fase 6.
- Notificaciones asociadas a cada transición → depende de `PA-031`.
- Representación visual del flujo en pantalla.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `estado.clave` | Sí | Único; pertenece al catálogo declarado | No |
| `estado.es_final` | Sí | Verdadero o falso | No |
| `transicion_valida.origen` / `destino` | Sí | Estados declarados; el destino no puede ser el origen | No |
| `transicion_valida.permiso` | Sí | Permiso del catálogo (`HU-003`) | No |
| `transicion_valida.exige_motivo` | Sí | Verdadero o falso | No |
| `transicion.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `transicion.actor_id` y `actor_tipo` | Sí | `usuario` \| `sistema` \| `contraparte` | No |
| `transicion.ocurrida_en` | Sí | Momento; se escribe una sola vez | No |
| `transicion.motivo` | Condicional | Obligatorio si la transición lo exige | No |
| `transicion.version_configuracion_id` | Sí | Versión vigente al ejecutarla | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: §38, Fase 3, Fase 18
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (máquina de estados en la base de
  datos), `ADR-0005` (las transiciones son eventos)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-029` — bloqueante. Si el Oficial de Cumplimiento puede decidir con
  requisitos pendientes, hace falta una transición de excepción registrada hacia "pendiente de
  decisión"; si no puede, hace falta un bloqueo duro. Son dos máquinas distintas. **No pasa de
  `borrador` hasta que se responda.** `PA-028` — qué transición corresponde cuando el enlace
  expira sin completarse.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-003` (permisos por transición), `HU-006` (bitácora), `HU-002`.
- **Habilita a:** todo el recorrido de la épica. Ninguna historia posterior mueve un expediente
  por su cuenta.
