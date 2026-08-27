---
id: ADR-0001
titulo: Stack técnico para el piloto y la primera versión comercial
estado: propuesto
fecha: 2026-08-21
reemplaza-a: —
reemplazado-por: —
---

# ADR-0001 — Stack técnico

## Contexto

> **Enmienda del 2026-08-21.** El cliente definió el modelo de licencias (cupo mensual de
> consultas incluido, con excedente pagado por consumo o mediante cambio de plan) y decidió
> que **el cobro entra al piloto**. Ver [`ADR-0002`](ADR-0002-pasarela-de-pagos-y-facturacion.md).
> El catálogo de librerías concretas está en
> [`08-desarrollo/librerias-y-entorno-ia.md`](../08-desarrollo/librerias-y-entorno-ia.md), y el
> detalle de cómo funciona el backend sin servicio separado en
> [`08-desarrollo/arquitectura-de-aplicacion.md`](../08-desarrollo/arquitectura-de-aplicacion.md).


Restricciones reales del proyecto en el momento de decidir. Son ellas, y no las
preferencias tecnológicas, las que mandan sobre esta decisión:

| Restricción | Valor |
|---|---|
| Equipo | **Un desarrollador**, apoyado en herramientas de IA |
| Plazo al piloto | **20 semanas** con cliente real (revisado el 2026-08-21 al no recortar alcance) |
| Presupuesto pre-ingresos | **< 100 USD/mes** en infraestructura y APIs |
| Escala a 12 meses | < 100 organizaciones cliente |
| Documentos | PDF mayormente digital, formatos fijos colombianos, extracción **síncrona** |
| Residencia de datos | EE.UU. es aceptable (`SUP-004`) |
| Requisito duro | **Auditoría inmutable y retención larga** |
| Enterprise | Sin SSO ni instancia aislada por ahora |

Naturaleza del producto (ver `07-integraciones/README.md`): el valor no está en el CRUD
sino en **el dato consolidado de listas restrictivas y en la calidad del cruce**. El
costo operativo del cliente no es encontrar coincidencias: es **descartarlas con
evidencia defendible**.

De todas las restricciones, **"un solo desarrollador" es la que más decide**. Un segundo
lenguaje, un segundo despliegue o una capa de API adicional son impuestos que se pagan
todas las semanas.

## Decisiones

### Núcleo

1. **TypeScript de punta a punta.** Un solo lenguaje, tipos compartidos entre servidor y
   cliente, un solo ecosistema de herramientas.
2. **Next.js (App Router) como monolito**, desplegado en **Vercel** (plan Pro, 20 USD/mes).
   Un repositorio, un despliegue, sin backend separado.
3. **Sin GraphQL. Sin API REST interna.** Para que el frontend hable con su propio backend
   se usan **Server Components y Server Actions** con tipos compartidos: seguridad de tipos
   sin capa intermedia. Una **API REST pública versionada** se creará solo cuando un tercero
   deba consumirla, como superficie pequeña y deliberada.
4. **Supabase (plan Pro, 25 USD/mes, región `us-east-1`)** para Postgres, Auth y Storage.
   *Nota:* el plan Free queda descartado porque pausa proyectos inactivos y no incluye
   backups, incompatible con el requisito de auditoría y retención.
5. **Multi-tenant con Row Level Security.** Ver decisión de modelo de identidad abajo.
6. **Drizzle ORM.** Se prefiere sobre Prisma por su cercanía al SQL: el proyecto necesita
   políticas RLS, CTEs recursivas para el grafo de vinculaciones y control fino de consultas.

### Modelo de identidad y pertenencia

7. **Todo es una organización desde el día uno.** El usuario individual no es un caso
   especial: es una organización de un miembro con un plan de una licencia. El paso de
   persona a empresa es un cambio de plan y una invitación, nunca una migración de datos.
8. **Jerarquía:** `Organización → Proyecto/Workspace → Contrapartes y consultas`.
9. **Membresía N:M** entre usuario y organización (una persona puede pertenecer a varias),
   con rol por membresía. Registrado como `PA-013` mientras el cliente no lo confirme.
10. **Cada tabla del dominio lleva `organization_id` y política RLS**, sin excepciones.

### Documentos y extracción

11. **Subida directa a Supabase Storage con URL firmada.** El archivo nunca viaja por la
    API: Vercel limita el body de request y response a **4.5 MB**.
12. **Extracción modelada como job desde el inicio**, aunque en el piloto se ejecute de
    forma síncrona mientras el usuario espera. Estado persistido, reintentable, idempotente.
13. **Estrategia híbrida de extracción:** plantillas deterministas para el núcleo de
    formatos colombianos conocidos (RUT, cámara de comercio, cédula), y **LLM multimodal
    como respaldo** para la cola larga. El motor se aísla tras un puerto reemplazable.
14. Librerías: `unpdf` / `pdfjs-dist` para texto de PDF digital; **Vercel AI SDK** para la
    capa de LLM, con proveedor intercambiable.

### Screening y agente

15. **OpenSanctions API SaaS** como fuente internacional consolidada (0,10 €/llamada;
    ver `07-integraciones/README.md`). **No auto-hospedar yente en esta fase.**
16. **Un proveedor local colombiano por cotizar** para Procuraduría, Contraloría, Policía
    y RUES (`PA-012`).
17. **El agente de IA propone y justifica; el humano firma.** El agente descarta ruido,
    prioriza y redacta la justificación con evidencia, pero toda decisión queda registrada
    a nombre de una persona. Es la postura más defendible ante un supervisor y la que
    limita la responsabilidad.
18. **Cada screening se persiste como evento inmutable** con: snapshot del resultado (no un
    enlace), versión del dataset consultado, modelo y prompt usados, costo de la llamada, y
    quién firmó la decisión. Esta tabla es simultáneamente la evidencia de auditoría y el
    instrumento de medición de consumo.

### Interfaz

19. **Tailwind CSS + shadcn/ui.** Componentes sobre los que se tiene el código, no una
    dependencia opaca; además es lo mejor soportado por asistentes de IA.
20. **React Hook Form + Zod** para los formularios de captura, que son la pieza de React
    más compleja del producto.
21. **Zod como esquema único** de validación, compartido entre cliente y servidor.
22. **TanStack Query solo donde exista estado de servidor en el cliente** (analítica,
    listados con filtros). No por defecto: con RSC la mayoría de las pantallas no lo necesita.
23. **Visualización:** `Recharts` para gráficos; **React Flow** para el grafo de vinculaciones
    entre entidades, socios y beneficiarios finales.
24. **Reportes exportables:** `@react-pdf/renderer` para PDF y `exceljs` para Excel,
    generados en el servidor.

### Operación

25. **Sentry** (plan gratuito) para errores y trazas.
26. **Vitest** para lógica y **Playwright** para los flujos críticos. **Las pruebas de
    aislamiento RLS son obligatorias** y se escriben junto con las políticas, no después.
27. **Un solo repositorio**, sin monorepo. No hay nada que compartir todavía.

## Consecuencias

**A favor**

- Un solo lenguaje, un despliegue y un proveedor de datos: la superficie operativa mínima
  posible para una persona sola.
- Costo fijo de arranque de ~45 USD/mes, dentro del presupuesto.
- Postgres cubre a la vez el dominio, la auditoría, el grafo de relaciones y la analítica.
  A menos de 100 organizaciones no hace falta nada más.
- Es el stack mejor representado en los asistentes de IA, lo que multiplica la velocidad
  del único desarrollador.

**Costo que se asume**

- **Acoplamiento a Vercel y Supabase.** Mitigación: mantener la lógica de dominio fuera de
  los handlers y detrás de puertos, para que migrar sea reescribir bordes y no el núcleo.
- **Aprender React "de verdad" queda concentrado** en formularios y analítica; los Server
  Components esconden buena parte del React clásico. Es una pérdida consciente frente al
  plazo de 12 semanas.
- **El costo variable de screening no está acotado** hasta cotizar al proveedor local y
  decidir el monitoreo continuo (`PA-011`, `PA-012`).
- **Sin SSO empresarial.** Supabase Auth no lo cubre cómodamente; si aparece un cliente que
  lo exija, será un ADR nuevo (candidatos: Clerk, WorkOS).

## Skills que hay que dominar

Ordenadas por riesgo real para el proyecto, no por dificultad:

1. **Postgres RLS y modelo de permisos.** El fallo aquí es una fuga de datos entre clientes.
   Innegociable.
2. **SQL más allá del CRUD:** CTEs recursivas para el grafo, agregados para la analítica,
   índices y `pg_trgm` para búsqueda por nombre.
3. **Matching difuso de nombres.** El núcleo técnico real: dos apellidos, homónimos, alias
   y transliteraciones. Hay que saber medir precisión y exhaustividad, no solo "que funcione".
4. **Next.js App Router:** frontera servidor/cliente, caché y revalidación. Es donde más se
   equivoca quien viene de React clásico.
5. **Diseño de sistemas con LLM:** grounding, evaluación, control de costos y trazabilidad
   de prompts. Un agente sin medición de precisión no es defendible ante un supervisor.
6. **Modelo de suscripciones y medición de consumo:** asientos, cuotas y atribución de costo.
7. **React aplicado:** formularios controlados complejos y rendimiento en vistas de datos.

## Trabajo con asistentes de IA

El stack se eligió también por esto, y conviene explotarlo deliberadamente:

- **Convenciones en `CLAUDE.md`** en cada repo, para que el asistente respete estructura,
  nombres y patrones sin recordárselo.
- **El esquema de Drizzle como fuente única de verdad**: de ahí se derivan tipos, validadores
  y formularios, y el asistente trabaja sobre un contrato explícito.
- **MCP de Supabase y de Vercel** para inspeccionar esquema, migraciones y logs sin salir del editor.
- **Precaución deliberada:** las políticas RLS, la lógica de cumplimiento y el cálculo de
  riesgo son justo donde el código generado por IA es más peligroso, porque *parece*
  correcto. Todo eso va con pruebas escritas a mano.

## Alternativas descartadas

| Alternativa | Por qué no |
|---|---|
| Backend separado (NestJS, .NET, FastAPI) | Segundo despliegue y segundo contexto mental para una sola persona con 12 semanas. |
| GraphQL | Su beneficio aparece con varios equipos y varios consumidores; su costo lo pagaría íntegro un solo desarrollador. |
| Base de datos de grafos (Neo4j) | Postgres con CTEs recursivas sobra a esta escala. Sería infraestructura y costo sin beneficio. |
| Auto-hospedar yente (OpenSanctions) | De días a semanas de ingeniería más Elasticsearch. Correcto dentro de un año con volumen; letal hoy. |
| Clerk / WorkOS para autenticación | Resuelven SSO empresarial, que hoy nadie exige. Supabase Auth viene incluido en un servicio que ya se paga. |
| Construir la ingesta propia de listas | Es un producto completo en sí mismo, con mantenimiento permanente. No se compite ahí. |
| Vercel plan Hobby | Es de uso no comercial: con un cliente pagando se estaría fuera de términos. |

## Decisiones aplazadas

- ~~**Pasarela de pagos**~~ — **ya no se aplaza.** El cliente decidió incluir el cobro en el
  piloto y definió el modelo de licencias (cupo de consultas incluido + excedente por consumo).
  Resuelto en [`ADR-0002`](ADR-0002-pasarela-de-pagos-y-facturacion.md). El alcance del piloto
  se mantiene completo y lo que se ajusta es el calendario: 20 semanas en vez de 12, según
  `02-producto/roadmap.md`.
- **Auto-hospedaje del índice de screening** — se reevalúa cuando el monitoreo continuo o
  el volumen lo justifiquen. Será `ADR-0003`.
- **SSO empresarial** — cuando exista un contrato que lo pague.
