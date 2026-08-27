---
id: HU-002
titulo: Aislamiento entre organizaciones con contexto de usuario
estado: borrador
epica: EP-000
prioridad: Must
actualizado: 2026-08-27
---

# HU-002 — Aislamiento entre organizaciones con contexto de usuario

## Historia

**Como** Sistema
**quiero** que el filtro por organización cliente lo aplique la base de datos a partir del
contexto de usuario propagado en cada transacción, y no el código de la aplicación
**para** eliminar la posibilidad de que un olvido en una consulta exponga los datos de un
cliente del SaaS a otro.

## Contexto

La §31 es explícita: cada cliente tiene usuarios, expedientes, configuración, claves y
bitácora propios, y los datos de una contraparte **no se reutilizan entre clientes del SaaS**.
Aunque dos organizaciones clientes conozcan al mismo proveedor, son dos expedientes
independientes.

La trampa está descrita en `08-desarrollo/arquitectura-de-aplicacion.md`: si la aplicación se
conecta a Postgres con credenciales de administrador, **las políticas de aislamiento no se
aplican** —esa conexión las omite por diseño— y la única protección pasa a ser acordarse de
filtrar por `organization_id` en cada consulta. Tarde o temprano se olvida una, y el fallo no
se manifiesta como un error sino como un cliente viendo lo ajeno.

Por eso la decisión ya tomada es la opción B: **cada transacción propaga la identidad del
usuario y la política de la base de datos decide**. Esta historia la implementa y, sobre todo,
la prueba.

## Criterios de aceptación

```gherkin
Escenario: Una consulta con contexto de usuario solo devuelve lo de su organización cliente
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Y filas de una tabla del dominio pertenecientes a "Alfa Ficticia S.A.S." y a "Beta Ficticia S.A.S."
  Cuando el usuario consulta esa tabla con su contexto de usuario propagado
  Entonces obtiene únicamente las filas de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna fila de "Beta Ficticia S.A.S."
```

```gherkin
Escenario: Escribir en otra organización cliente es rechazado por la base de datos
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando intenta insertar o modificar una fila con el identificador de "Beta Ficticia S.A.S."
  Entonces la operación es rechazada por la política de la base de datos
  Y el rechazo ocurre aunque la consulta no incluya ningún filtro por organización cliente en el código
```

```gherkin
Escenario: Sin contexto de usuario no se ve nada
  Dado una conexión de aplicación sin contexto de usuario propagado
  Cuando se consulta cualquier tabla del dominio
  Entonces no se devuelve ninguna fila
  Y el resultado vacío no depende de que la consulta llevara un filtro por organización cliente
```

```gherkin
Escenario: Prueba de aislamiento obligatoria por tabla
  Dado una tabla nueva del dominio incorporada al esquema
  Cuando se ejecuta la batería de pruebas de aislamiento
  Entonces existe una prueba automatizada para esa tabla que verifica lectura y escritura cruzadas entre dos organizaciones clientes
  Y la batería falla si alguna tabla del dominio no tiene su prueba
  Y la batería falla si alguna tabla del dominio carece de la columna de organización cliente o de su política
```

```gherkin
Escenario: La conexión de administrador está acotada y deja rastro
  Dado un proceso de sistema autorizado a usar la conexión de administrador
  Cuando ejecuta una operación que omite las políticas de aislamiento
  Entonces esa operación solo puede invocarse desde el módulo de acceso privilegiado
  Y queda registrada en la bitácora identificada como proceso automático
  Y ningún camino de la aplicación que atienda a un usuario puede alcanzar esa conexión
```

## Reglas de negocio

- **Toda tabla del dominio lleva `organization_id` y su política de aislamiento, sin
  excepciones** (`ADR-0001`). La única excepción admitida es la tabla de cuentas de usuario de
  `HU-001`, que vive por encima de las organizaciones clientes.
- El acceso a datos se hace por un único punto que abre la transacción con el contexto de
  usuario. **No debe existir forma de abrir una consulta del dominio sin contexto**; si no hay
  contexto, no hay filas.
- La conexión de administrador queda reservada a operaciones de sistema explícitas
  —migraciones, trabajos programados, conciliación— en un módulo aparte y con nombre incómodo
  a propósito, para que su uso sea siempre una decisión consciente.
- Las pruebas de aislamiento se escriben **junto con la política, no después** (`ADR-0001`).
  Una tabla del dominio sin prueba de aislamiento no está terminada.
- Los datos de una contraparte no se reutilizan entre organizaciones clientes (§31). Si dos
  clientes conocen a la misma contraparte, son dos registros y dos expedientes, y se paga dos
  veces la consulta externa.

## Fuera de alcance

- El acceso de la contraparte por enlace, que propaga un expediente en lugar de un usuario. Es
  un segundo modo del mismo mecanismo y llega con la Fase 1.
- Permisos por rol dentro de la organización cliente: el aislamiento decide **de qué
  organización** son las filas; el rol decide **qué puede hacer** el usuario con ellas →
  `HU-003`.
- Cifrado en reposo, gestión de claves y límites de petición.
- Tablas concretas del expediente: aquí se define la regla y su prueba, no el esquema del
  expediente.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `organization_id` (en toda tabla del dominio) | Sí | Organización cliente existente; inmutable una vez escrito | No |
| Contexto de transacción: usuario | Sí | Cuenta existente y activa | No |
| Contexto de transacción: organización cliente | Sí | Debe corresponder a una membresía activa del usuario (`HU-001`) | No |
| Contexto de transacción: tipo de actor | Sí | `usuario` \| `sistema` — el valor `contraparte` se reserva para la Fase 1 | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §31
- Decisiones: `ADR-0001` (puntos 5, 10 y 26), `08-desarrollo/arquitectura-de-aplicacion.md`
  (sección de acceso a datos y aislamiento)
- Requisitos: alimenta la categoría de aislamiento de `03-requisitos/no-funcionales.md`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna bloquea el diseño. Queda en `borrador` porque depende de
  `HU-001`, que sí está bloqueada por `PA-024`.
- **Supuestos:** `SUP-002` (multiempresa, confirmado §31).
- **Depende de:** `HU-001`.
- **Habilita a:** todas las demás historias de la plataforma. Ninguna tabla del dominio puede
  escribirse antes de que esta historia esté terminada.
- **Riesgo:** es la historia cuyo fallo es más caro y menos visible. Un error aquí no produce
  una excepción, produce una fuga silenciosa. De ahí que el criterio de aceptación exija que la
  batería de pruebas **falle sola** cuando aparezca una tabla sin política, en vez de confiar
  en que alguien lo revise.
