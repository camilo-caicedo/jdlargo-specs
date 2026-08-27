---
id: HU-024
titulo: Verificación de datos contra fuentes externas
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-024 — Verificación de datos contra fuentes externas

## Historia

**Como** Analista de Cumplimiento
**quiero** que el sistema contraste lo que declaró la contraparte contra los registros oficiales
y me muestre en qué difieren
**para** saber qué parte del expediente está respaldada por una fuente autorizada y qué parte
sigue siendo solo la palabra de la contraparte.

## Contexto

Es la Fase 11 y el punto donde aparece el tercer origen de `ADR-0005`: **lo verificado**. Hasta
aquí el expediente contenía lo declarado y lo extraído; ninguno de los dos es confirmación.

La Fase 11 exige además un cuarto estado que suele olvidarse: **dato no verificable**. No es lo
mismo un dato que ninguna fuente confirma que un dato que nadie ha intentado verificar todavía, y
la plataforma nunca los presenta como equivalentes.

La §34 añade un límite: **no depender de una sola fuente para verificar identidad**. Cuántas
fuentes se exigen para dar por verificado un campo es configuración del cliente, no una regla
del producto.

## Criterios de aceptación

```gherkin
Escenario: Verificar un campo contra una fuente
  Dado un campo declarado y una fuente del catálogo asociada a ese campo por la matriz
  Cuando el sistema consulta la fuente
  Entonces se registra una afirmación con origen verificado, citando la fuente y la consulta
  Y la respuesta de la fuente queda congelada como evidencia, no como un enlace a ella
  Y quedan registrados la fecha, la hora y el costo de la consulta
```

```gherkin
Escenario: Lo verificado difiere de lo declarado
  Dado un campo con una afirmación declarada
  Cuando la verificación devuelve un valor distinto
  Entonces existen ambas afirmaciones, cada una con su origen
  Y se abre una discrepancia sobre ese campo
  Y el sistema no elige ninguna por su cuenta
```

```gherkin
Escenario: Un dato que ninguna fuente puede confirmar
  Dado un campo para el que la configuración no asocia ninguna fuente capaz de confirmarlo
  Cuando se consulta su estado de verificación
  Entonces aparece como no verificable
  Y se distingue de un dato pendiente de verificar y de uno verificado
```

```gherkin
Escenario: La fuente no responde
  Dado una fuente indisponible en el momento de la consulta
  Cuando se intenta verificar un campo contra ella
  Entonces el campo queda como pendiente de verificar
  Y queda registrado el intento fallido con su momento
  Y no se registra ninguna afirmación verificada
```

```gherkin
Escenario: La evidencia sobrevive al proveedor
  Dado una verificación ejecutada hace tiempo
  Cuando se consulta su evidencia
  Entonces se obtiene la respuesta tal como llegó en ese momento
  Y esa evidencia sigue siendo legible aunque el proveedor haya cambiado o desaparecido
```

```gherkin
Escenario: Un solo respaldo no basta cuando la configuración exige más
  Dado una configuración que exige dos fuentes para dar por verificada la identidad
  Cuando solo una de ellas confirma el dato
  Entonces el campo no aparece como verificado
  Y se indica qué respaldo falta
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las verificaciones
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta verificaciones con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Una verificación produce una **afirmación de origen `verificado`** (`HU-005`) que cita la
  fuente, la consulta y su evidencia.
- La evidencia se guarda **congelada**: la respuesta tal como llegó, con la versión del conjunto
  de datos consultado, no un enlace ni una referencia externa (`ADR-0001` §18).
- La verificación **no cierra** una discrepancia ni la evita: si difiere de lo declarado, se abre
  una (`HU-019`) y la resuelve una persona.
- Cuántas fuentes se exigen para dar un campo por verificado es **configuración** del cliente. La
  §34 prohíbe depender de una sola para identidad.
- Un dato **no verificable** es un estado propio y se muestra como tal. No se presenta como
  verificado ni se oculta.
- Cada consulta registra su costo en el mismo evento que su evidencia, para auditoría y para
  medición de consumo.
- La consulta ocurre **siempre en el servidor**: las credenciales nunca llegan al navegador.
- Los reintentos usan clave de idempotencia: un reintento no vuelve a cobrar ni a consultar.

## Fuera de alcance

- El screening contra listas, que tiene su propia mecánica → `HU-025`.
- La comparación de nombres → `HU-026`.
- La re-verificación periódica → Fase 6.
- El control de cupo antes de gastar → Fase C. Aquí se registra el costo; bloquear por cuota es
  otra épica.
- La verificación de personas relacionadas → Fase 4.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `verificacion.organization_id` | Sí | Organización cliente existente | No |
| `verificacion.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `verificacion.campo` | Sí | Campo definido en la versión de configuración citada | No |
| `verificacion.fuente_id` | Sí | Fuente del catálogo (`HU-023`) | No |
| `verificacion.ejecutada_en` | Sí | Momento; se escribe una sola vez | No |
| `verificacion.resultado` | Sí | `confirmado` \| `difiere` \| `sin_datos` \| `no_disponible` | Sí |
| `verificacion.evidencia` | Sí | Respuesta congelada de la fuente | Sí |
| `verificacion.version_datos` | No | Versión del conjunto de datos consultado, si la fuente la reporta | No |
| `verificacion.costo` | Sí | Costo de la consulta; puede ser cero | No |
| `verificacion.clave_idempotencia` | Sí | Impide cobrar dos veces el mismo reintento | No |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: Fase 11, §2, §34, §44 preguntas E y F
- Decisiones: `ADR-0005` (lo verificado es un origen), `ADR-0001` (evidencia congelada con
  versión del conjunto de datos y costo)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-005` (qué fuentes), `PA-012` (costo del proveedor local).
  **Queda en `borrador`.**
- **Supuestos:** `SUP-003`.
- **Depende de:** `HU-023` (catálogo), `HU-012` (hay algo declarado que verificar), `HU-005`,
  `HU-019` (las diferencias abren discrepancia).
- **Habilita a:** el motor de riesgo de la Fase 4, que pondera qué está verificado y qué no.
- **Riesgo:** guardar un enlace a la respuesta en vez de la respuesta. El día que el proveedor
  cambie su formato o cierre, el expediente se queda sin evidencia y no hay forma de recuperarla.
