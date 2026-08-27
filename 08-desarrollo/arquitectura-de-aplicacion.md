---
id: DEV-arquitectura
estado: propuesto
actualizado: 2026-08-21
---

# Arquitectura de la aplicación

Complemento de `ADR-0001`, que decidió un monolito Next.js sin backend separado. Este
documento responde la pregunta obvia: **entonces, ¿dónde ocurren las consultas a la base de
datos y las llamadas a servicios externos?**

> **Ampliado el 2026-08-25** con lo que exige el documento funcional del cliente: el portal
> externo de la contraparte, la máquina de estados del expediente, la firma electrónica por
> niveles, el motor de reglas (`ADR-0004`) y el modelo de procedencia (`ADR-0005`).

## "Sin backend dedicado" no es "sin backend"

El backend existe. Lo que no existe es un **servicio separado**, con su propio repositorio,
su propio despliegue y su propio contrato HTTP.

Cuando se escribe un Server Component o un Server Action, ese código **no viaja al
navegador**: se ejecuta en Node, dentro de una función en Vercel. Ahí viven la conexión a
Postgres, las llaves de OpenSanctions, Wompi y Alegra, y toda la lógica de negocio. El
navegador recibe HTML ya renderizado o el resultado de una acción, nunca el acceso.

La diferencia con un backend tradicional es de **ceremonia, no de arquitectura**. En el
modelo clásico se escribe un endpoint, un cliente HTTP, tipos duplicados a lado y lado, y
serialización en el medio. Aquí se llama una función tipada que resulta ejecutarse del otro
lado. Mismo backend, sin el trámite.

## Dónde corre cada cosa

| Necesidad | Dónde vive | Notas |
|---|---|---|
| Leer datos para pintar una pantalla | **Server Component** | Consulta directa con Drizzle. Sin endpoint, sin `useEffect`, sin estado de carga. |
| Crear, editar o borrar desde la interfaz | **Server Action** | Por debajo es un POST al servidor, pero se invoca como una función tipada. |
| Llamar a OpenSanctions, Wompi o Alegra | **Siempre servidor** | Las llaves nunca llegan al navegador. Sin excepciones. |
| Recibir un webhook de Wompi | **Route Handler** (`app/api/webhooks/wompi/route.ts`) | Endpoint HTTP real: lo llama un tercero. |
| Tareas programadas (cierre de ciclo, monitoreo) | **`pg_cron` + `pgmq`** dentro de Postgres, o Route Handler protegido invocado por cron | Sin infraestructura adicional. |
| API pública para clientes que integren | **Route Handlers versionados** (`app/api/v1/...`) | Solo cuando exista un tercero que la consuma. Hoy no existe. |
| Interactividad pura (filtros, formularios, gráficos) | **Client Component** | Lo único que corre en el navegador. |

## La estructura que hace esto reversible

El riesgo real de un monolito no es el rendimiento: es que la lógica de negocio se derrame
dentro de los componentes y quede imposible de extraer. Se evita con una regla de carpetas:

```
src/
  app/                    Rutas, páginas y Route Handlers — ADAPTADORES DELGADOS
  server/                 ← ESTO ES EL BACKEND
    organizaciones/       Cada módulo: casos de uso + repositorio + tipos
    contrapartes/
    screening/            Puertos hacia proveedores externos
    consumo/              Cuotas, medición y ciclo de facturación
    documentos/
    db/                   Esquema Drizzle, cliente y migraciones
  components/             Interfaz
```

**Un Server Action no contiene lógica de negocio.** Valida su entrada con Zod, llama a un
caso de uso en `src/server/`, y traduce el resultado a algo que la interfaz entienda. Si un
Server Action tiene más de veinte líneas, la lógica está en el lugar equivocado.

El beneficio concreto: el día que haya que extraer un backend real, se mueve `src/server/`
y se reescriben solo los bordes. Es lo que mantiene reversible la decisión de `ADR-0001`.

## Tres recorridos reales

**1. Listar las contrapartes de un proyecto.** El Server Component pide los datos a
`server/contrapartes`, que consulta Postgres con Drizzle en la misma función de servidor.
Se renderiza el HTML y se envía. Cero peticiones desde el navegador, cero endpoints.

**2. Ejecutar un screening.** El usuario aprieta un botón en un Client Component, que invoca
un Server Action. El Action valida la entrada y llama al caso de uso, que en una sola
transacción: verifica la cuota disponible, llama a OpenSanctions, persiste la evidencia
inmutable con su snapshot, registra el costo y descuenta el consumo. Devuelve el resultado
a la interfaz. **La cuota se verifica antes de gastar y todo ocurre en una transacción**, para
que un fallo a mitad de camino no deje consumo cobrado sin evidencia guardada.

**3. Recibir la confirmación de un pago.** Wompi llama al Route Handler. Este verifica la
firma, guarda el evento con su clave de idempotencia —de modo que un reintento de Wompi no
lo procese dos veces— y delega en `server/consumo` para conciliar la factura.

## Acceso a datos y RLS — la decisión que más importa

Aquí hay una trampa que conviene ver antes de escribir la primera migración.

Si Drizzle se conecta a Postgres con credenciales de administrador (la llave de servicio de
Supabase), **las políticas RLS no se aplican**: esa conexión las omite por diseño. La única
protección contra fuga entre organizaciones pasaría a ser acordarse de filtrar por
`organization_id` en cada consulta. Tarde o temprano se olvida una, y el fallo no se
manifiesta como un error sino como un cliente viendo datos de otro.

| Opción | Cómo funciona | Riesgo |
|---|---|---|
| **A · Conexión de administrador** | Se omite RLS; el filtro por organización vive en el código | Un olvido = fuga de datos entre clientes |
| **B · Conexión con contexto de usuario** | Cada transacción propaga la identidad del usuario, y RLS aplica en la base de datos | Un poco más de ceremonia |

**Decisión: opción B para todas las tablas del dominio.** El filtro por organización es
responsabilidad de la base de datos, no de la memoria del desarrollador. Se encapsula en un
único helper de acceso a datos, de modo que sea imposible abrir una consulta sin contexto.

La conexión de administrador queda reservada para operaciones de sistema explícitas
—migraciones, jobs, conciliación de pagos— en un módulo aparte y con nombre incómodo a
propósito, para que su uso sea siempre una decisión consciente.

> Esto es lo que hace que la prueba de aislamiento de cada tabla (`ADR-0001` §26) sea
> realmente una prueba: verifica la política, no el recuerdo de haber filtrado.

## Conexiones a la base de datos

Las funciones serverless se crean y destruyen constantemente. Con conexión directa a
Postgres se agota el límite de conexiones rápidamente.

**Usar siempre el pooler de Supabase en modo transacción**, no la conexión directa. La
conexión directa se reserva para migraciones, que son las únicas que necesitan sesión
persistente. Es un detalle de una línea de configuración que, mal puesto, se manifiesta
como caídas intermitentes bajo carga y cuesta días diagnosticar.

## Manejo de errores

- **Errores de dominio tipados** (cuota agotada, proveedor caído, documento ilegible) que la
  interfaz sepa traducir a un mensaje útil. No excepciones genéricas.
- **`error.tsx` y `not-found.tsx`** de Next.js para los límites de error de cada ruta.
- **Reintentos con retroceso exponencial** (`p-retry`) solo contra proveedores externos, y
  **siempre con clave de idempotencia**: un reintento no puede volver a cobrar ni a consultar.
- **Sentry** captura tanto el error del servidor como el del navegador con la misma traza.

## El portal de la contraparte — una segunda superficie, con otras reglas

La Fase 4 introduce algo que no estaba contemplado en `ADR-0001`: **la contraparte no tiene
cuenta**. Entra con un enlace y un token, y aun así carga documentos de identidad y datos
financieros. Es una superficie pública, y hay que tratarla como tal.

| Aspecto | Aplicación interna | Portal de contraparte |
|---|---|---|
| Identidad | Sesión de Supabase Auth | Token de acceso con expiración, sin usuario |
| Segundo factor | MFA opcional del usuario | **OTP por correo o SMS** cuando el riesgo lo justifique |
| Alcance de datos | Toda su organización, según rol | **Un único expediente**, nada más |
| Aislamiento | RLS por organización | RLS por expediente, derivada del token |
| Límite de peticiones | Bajo | **Estricto** — es el punto expuesto |

Decisiones que se derivan:

- El token es de un solo expediente, con expiración configurable, revocable, y **cada uso
  queda registrado con fecha, hora, IP y dispositivo** (Fase 4).
- El contexto de base de datos que se propaga (`ADR-0001`, sección de RLS) no es un usuario
  sino un expediente. Es un segundo modo del mismo mecanismo, no una excepción a él.
- El OTP por SMS implica un proveedor y un costo por mensaje que hoy no está presupuestado
  (`PA-019`). Por correo se resuelve con el proveedor de correo ya elegido.
- Toda la ruta del portal vive bajo un segmento propio del enrutador, con su propio
  middleware. Nunca comparte capa de acceso con la aplicación interna.

## La máquina de estados del expediente

El documento define estados explícitos para el expediente (§38) y para cada documento
(Fase 7). Se implementan como una **máquina de estados en la base de datos**, no como una
columna de texto que cada parte del código actualiza a su antojo:

- Las transiciones válidas son datos, y una transición inválida es un error de dominio.
- Cada transición se persiste con quién la provocó, cuándo y por qué. La bitácora del
  expediente **es** la lista de sus transiciones.
- No hace falta una librería de máquinas de estados en el cliente: el estado vive en el
  servidor y la interfaz solo lo refleja.

## Firma electrónica — tres niveles, dos construibles

La Fase 19 exige niveles configurables por el cliente:

| Nivel | Qué es | Cómo se implementa |
|---|---|---|
| 1 | Aceptación electrónica con evidencia | Propio: hash del documento, IP, fecha y hora, versión aceptada |
| 2 | Firma reforzada con factor adicional | Propio: nivel 1 más OTP verificado |
| 3 | Firma digital certificada | **Proveedor externo acreditado** (`PA-020`) |

Los niveles 1 y 2 se construyen y comparten la misma estructura de evidencia. El nivel 3 es
una integración con un tercero, con costo por firma, y **no entra en el primer MVP**.

La capa de firma se diseña detrás de un puerto único, para que el nivel 3 se enchufe después
sin tocar el flujo del expediente.

## Dónde vive el motor de reglas

Siguiendo `ADR-0004`, la configuración de cumplimiento es un módulo de dominio propio:

```
src/server/
  cumplimiento/
    configuracion/    Estándares, matrices, metodologías — publicación y versionado
    evaluador/        Evalúa condiciones y produce acciones, con su explicación
    formularios/      Construye el esquema de validación desde la configuración
  expedientes/
    afirmaciones/     ADR-0005: el registro de procedencia
    estados/          Transiciones válidas
```

Regla dura: **el evaluador es una función pura.** Recibe la configuración vigente y los
datos del expediente, y devuelve resultado más explicación. No lee de la base de datos ni
escribe en ella. Es lo que lo hace comprobable con tablas de casos, que es la única forma
sensata de tener confianza en un motor de cumplimiento.

## Monitoreo continuo y trabajos programados

La Fase 20 y la Fase 21 están dentro del alcance que definió el cliente, lo que confirma la
elección de `pgmq` y `pg_cron` dentro de Postgres:

- **Vencimientos y renovaciones** — un trabajo diario que revisa vigencias y genera alertas.
- **Re-screening** — periódico según la metodología, más disparado por evento.
- **Recordatorios de expedientes sin completar** (§46) antes de que expire el enlace.

Cada uno genera, cuando corresponde, un **caso** (Fase 13), nunca una decisión automática.

> ⚠️ **Aviso de costo.** Que el monitoreo continuo esté en el alcance reactiva de lleno el
> riesgo descrito en `07-integraciones/README.md`. Y hay un agravante nuevo: la §31 prohíbe
> reutilizar datos de contrapartes **entre clientes del SaaS**, así que la deduplicación de
> consultas solo puede darse dentro de una misma organización. Si dos clientes verifican al
> mismo proveedor, se paga dos veces. Sin excepción.

## La IA y la transferencia internacional de datos

La §33 señala algo que `SUP-004` había cerrado demasiado rápido: usar un modelo alojado
fuera del país implica **enviar datos personales al exterior**, y eso hay que documentarlo
y sostenerlo contractualmente frente al cliente, que es el responsable del tratamiento.

Consecuencias de diseño:

- La capa de IA queda detrás de un puerto con proveedor intercambiable, ya previsto en
  `ADR-0001`, lo que ahora tiene una segunda justificación además de la técnica.
- Se **minimiza** lo que se envía al modelo: el fragmento necesario, no el expediente entero.
- Cada ejecución registra a qué proveedor se envió (§32), de modo que la pregunta "¿a dónde
  fueron estos datos?" tenga respuesta.
- Queda como `PA-021` confirmar con el cliente qué proveedores acepta y bajo qué contrato.

## Cuándo dejaría de servir este modelo

Vale la pena escribir las señales ahora, en frío, para no discutirlas después en caliente.
Si aparece alguna, se abre un ADR nuevo — no antes:

1. Un proceso que necesite correr más de los límites de duración de una función, de forma
   recurrente y no ocasional.
2. Un segundo consumidor real de la lógica de negocio: una app móvil, o clientes integrando
   por API con volumen serio.
3. Un equipo con más de tres o cuatro desarrolladores pisándose en el mismo repositorio.
4. Una necesidad de cómputo sostenido —indexación propia de listas, auto-hospedar `yente`—
   que no encaje en un modelo por invocación.

**Ninguna aplica hoy.** Y cuando alguna aplique, extraer `src/server/` será un trabajo de
días, no de meses, precisamente por la estructura de carpetas de arriba.
