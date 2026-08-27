---
id: HU-047
titulo: Medición de consumo y control del cupo
estado: borrador
epica: EP-007
prioridad: Must
actualizado: 2026-08-27
---

# HU-047 — Medición de consumo y control del cupo

## Historia

**Como** Sistema
**quiero** verificar el cupo disponible **antes** de ejecutar una consulta que cuesta dinero, y
registrar el consumo en la misma transacción que su evidencia
**para** que nunca haya un cargo sin evidencia guardada ni una evidencia sin su costo registrado.

## Contexto

Es el corazón de `ADR-0002` y su hallazgo central: **ningún proveedor de pagos colombiano tiene
motor de medición de consumo**. Se construye propio, sobre las tablas que el proyecto ya necesita
para auditoría —la misma fila que guarda la evidencia de un screening guarda su costo
(`ADR-0001` §18)—. Un solo activo, dos usos.

`08-desarrollo/arquitectura-de-aplicacion.md` describe el recorrido exacto: en una sola
transacción se verifica la cuota disponible, se llama al proveedor, se persiste la evidencia
inmutable, se registra el costo y se descuenta el consumo. **La cuota se verifica antes de gastar**,
para que un fallo a mitad de camino no deje consumo cobrado sin evidencia guardada.

Lo que falta decidir es qué ocurre al agotar el cupo: `ADR-0002` dice *"se bloquea o se marca como
excedente de forma explícita, jamás en silencio"* y deja las dos puertas abiertas. Es `PA-037`.

## Criterios de aceptación

```gherkin
Escenario: La cuota se verifica antes de gastar
  Dado una organización cliente con cupo disponible
  Cuando se solicita una consulta a una fuente externa
  Entonces primero se verifica el cupo
  Y solo entonces se llama al proveedor
  Y el consumo se registra en la misma transacción que la evidencia
```

```gherkin
Escenario: Un fallo no deja consumo sin evidencia
  Dado una consulta en curso
  Cuando falla después de llamar al proveedor pero antes de guardar la evidencia
  Entonces la transacción se deshace por completo
  Y no queda consumo registrado sin su evidencia
  Y el fallo queda registrado
```

```gherkin
Escenario: Un reintento no cobra dos veces
  Dado una consulta que se reintenta con la misma clave de idempotencia
  Cuando se ejecuta el reintento
  Entonces no se registra un consumo nuevo por la misma operación
  Y no se llama al proveedor otra vez si la consulta original ya se completó
```

```gherkin
Escenario: El cupo se agota
  Dado una organización cliente que llegó a su cupo del ciclo
  Cuando se solicita una consulta adicional
  Entonces el sistema aplica lo que defina la configuración de su plan
  Y la consulta se bloquea o se registra como excedente de forma explícita
  Y en ningún caso ocurre en silencio
```

```gherkin
Escenario: Aviso al acercarse al cupo
  Dado una organización cliente que alcanza el porcentaje de aviso configurado de su cupo
  Cuando se registra ese consumo
  Entonces se genera un aviso al Administrador
  Y el aviso indica cuánto queda y qué pasará al agotarse
```

```gherkin
Escenario: El evento de consumo es inmutable
  Dado un evento de consumo registrado
  Cuando se intenta modificarlo o eliminarlo
  Entonces la operación es rechazada por la base de datos
  Y el evento permanece idéntico
```

```gherkin
Escenario: El consumo se puede reconstruir consulta por consulta
  Dado un ciclo con consumo registrado
  Cuando se consulta su detalle
  Entonces se obtiene una fila por consulta, con su fuente, su momento, su costo de proveedor y su evidencia
  Y la suma de esas filas coincide con el total del ciclo
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el consumo
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando consulta su consumo con su contexto de usuario propagado
  Entonces obtiene únicamente el suyo
  Y ninguna consulta de una organización cliente descuenta cupo de otra
```

## Reglas de negocio

- **La cuota se verifica antes de gastar**, siempre. Nunca después.
- El consumo y su evidencia se registran **en la misma transacción**. No puede existir uno sin el
  otro.
- El evento de consumo es **inmutable** y es la misma fila que sirve de evidencia de auditoría
  (`ADR-0001` §18). Guarda organización cliente, momento, fuente, costo del proveedor y referencia
  a la evidencia.
- Toda consulta lleva **clave de idempotencia**: un reintento no vuelve a cobrar ni a consultar.
- Al agotarse el cupo, el comportamiento es el que defina el plan —bloquear o registrar excedente—
  y **nunca es silencioso** (`ADR-0002`, `PA-037`).
- Existe un aviso al acercarse al cupo, con el umbral configurado.
- El consumo de una organización cliente **jamás** afecta al cupo de otra (§31).
- El costo del proveedor se registra tal como es, para poder calcular margen: ingreso menos costo,
  por cliente y por ciclo.

## Fuera de alcance

- El cierre de ciclo y el cálculo del excedente a facturar → `HU-048`.
- El cobro → `HU-049`.
- La definición de los cupos → `HU-046` y `PA-036`.
- El consumo de la capa de IA, que también tiene costo por llamada: se registra en `HU-018`, y su
  facturación no está definida por el cliente.
- La optimización del consumo mediante reutilización de resultados entre organizaciones clientes:
  la §31 lo prohíbe.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `evento_consumo.organization_id` | Sí | Organización cliente existente | No |
| `evento_consumo.ciclo_id` | Sí | Ciclo abierto de esa organización | No |
| `evento_consumo.fuente_id` | Sí | Fuente del catálogo (`HU-023`) | No |
| `evento_consumo.evidencia_id` | Sí | Verificación o screening asociado | No |
| `evento_consumo.costo_proveedor` | Sí | Monto y moneda | No |
| `evento_consumo.ocurrido_en` | Sí | Momento; se escribe una sola vez | No |
| `evento_consumo.clave_idempotencia` | Sí | Única; impide doble cobro | No |
| `evento_consumo.dentro_de_cupo` | Sí | Verdadero o falso; los excedentes se marcan al registrarse | No |
| `plan.umbral_aviso` | Sí | Porcentaje del cupo a partir del cual se avisa | No |
| `plan.al_agotar_cupo` | Sí | `bloquear` \| `permitir_excedente` `(TBD — PA-037)` | No |

## Trazabilidad

- Épica: `EP-007`
- Capacidad: `CAP-07`
- Documento del cliente: §39
- Decisiones: `ADR-0002` (motor de medición propio), `ADR-0001` §18 (una tabla, dos usos),
  `08-desarrollo/arquitectura-de-aplicacion.md` (la cuota se verifica antes de gastar, todo en una
  transacción)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-037` — bloqueante: qué ocurre al agotar el cupo. `PA-036` — cuáles
  son los cupos. **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-023` (fuentes con costo), `HU-024` y `HU-025` (donde se consume), `HU-046`
  (planes).
- **Habilita a:** `HU-048`, `HU-049`, y responder `PA-011` y `PA-014` con datos reales en vez de
  con estimaciones.
- **Riesgo:** verificar la cuota después de consultar parece equivalente y no lo es: deja la
  puerta abierta a consumir sin cobrar o a cobrar sin evidencia. Es la clase de error que solo se
  descubre cuando ya hay dinero de por medio.
