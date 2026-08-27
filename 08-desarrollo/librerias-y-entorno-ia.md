---
id: DEV-librerias
estado: propuesto
actualizado: 2026-08-21
---

# Catálogo de librerías y entorno de trabajo con IA

Complemento operativo de `ADR-0001`. Dos principios:

1. **No se escribe lo que ya está resuelto y probado.** Sesiones, tokens, reintentos,
   formatos de moneda, generación de PDF: eso se instala.
2. **Sí se escribe lo que es del dominio.** Cuotas, evidencia, riesgo y matching son el
   producto. Delegarlos a una librería genérica es delegar el negocio.

Cuidado con la trampa intermedia: librerías que *parecen* ahorrar y en realidad esconden un
flujo que necesitas controlar. Autenticación casera y facturación casera están mal por
razones opuestas — la primera hay que comprarla, la segunda hay que construirla.

## Manejo de usuarios

| Necesidad | Librería | Notas |
|---|---|---|
| Registro, login, sesión, recuperación de cuenta, verificación de correo | `@supabase/supabase-js` + `@supabase/ssr` | Cubre todos los flujos de cuenta, incluido el refresco de sesión en el App Router. **No montar Auth.js/NextAuth encima**: duplicaría el modelo de sesión y es la causa número uno de bugs raros de autenticación en este stack. |
| Segundo factor (TOTP) | Supabase Auth (`auth.mfa`) | Viene incluido. No instalar nada. |
| **Organizaciones, membresías, invitaciones, roles** | **Ninguna — se construye** | No existe una buena librería para esto en Next + Supabase. Son tres tablas (`organizations`, `memberships`, `invitations`) con RLS. Aproximadamente una semana. La alternativa comprada es Clerk, que trae organizaciones nativas y quedó descartada en `ADR-0001`. |
| Autorización | Helpers tipados sobre el rol de la membresía | Empezar simple. Si los permisos se vuelven finos, entonces `@casl/ability`. No introducirlo el día uno. |
| Correos transaccionales (invitación, recuperación, aviso de cuota) | **Resend** + **React Email** | Plan gratuito suficiente para el piloto. Evita escribir plantillas HTML de correo a mano, que es trabajo ingrato y frágil. |
| Rate limiting de endpoints públicos | `@upstash/ratelimit` + Upstash Redis (plan gratuito) | Solo para abuso. **Las cuotas de negocio no van aquí**: van en Postgres, dentro de la transacción, porque tienen que ser exactas y auditables. |

## Formularios, validación y UI

| Necesidad | Librería |
|---|---|
| Esquema único de validación, cliente y servidor | `zod` |
| Formularios complejos de captura | `react-hook-form` + `@hookform/resolvers` |
| Estilos y componentes | `tailwindcss` + `shadcn/ui` (código propio, no dependencia opaca) |
| Iconos / notificaciones | `lucide-react` / `sonner` |
| Tablas con filtros, orden y paginación | `@tanstack/react-table` — el listado de contrapartes es exactamente este problema |
| Estado de servidor en cliente | `@tanstack/react-query`, **solo donde haga falta** |

## Datos

| Necesidad | Librería |
|---|---|
| ORM y migraciones | `drizzle-orm` + `drizzle-kit` |
| Búsqueda por nombre | Extensiones Postgres `pg_trgm` y `unaccent` — imprescindibles para nombres con tildes y para similitud |
| Archivos | Supabase Storage con URL firmada |

## Dinero, fechas y formato

El área donde más se rompen los sistemas de facturación, y donde más barato es hacerlo bien:

- **Nunca usar números de punto flotante para dinero.** Enteros en centavos, o `dinero.js`.
- `date-fns` + `@date-fns/tz`. Todo en UTC en base de datos, `America/Bogota` solo en presentación.
- `Intl.NumberFormat('es-CO')` para pesos: es nativo del lenguaje, no requiere librería.

## Pagos, facturación y documentos

| Necesidad | Solución |
|---|---|
| Wompi | Sin SDK oficial maduro para Node: cliente propio delgado sobre `ky`. Verificar firma de webhooks y guardar cada evento con clave de idempotencia. |
| Facturación DIAN | API REST de Alegra (ver `ADR-0002`) |
| Texto de PDF digital | `unpdf` o `pdfjs-dist` |
| Extracción con LLM | `ai` (Vercel AI SDK), con proveedor intercambiable |
| Exportables | `@react-pdf/renderer` (PDF) y `exceljs` (Excel) |

## Jobs, reintentos y resiliencia

- **`pgmq` + `pg_cron` dentro de Supabase**: cola y agendamiento en la misma base de datos,
  sin infraestructura adicional ni costo extra. Es lo que hace viable el requisito de
  "extracción como job" y el futuro monitoreo continuo.
- `p-retry` para reintentos con retroceso exponencial contra proveedores externos.
- Idempotencia por clave en tabla, para que un reintento nunca cobre ni consulte dos veces.

## Analítica

`recharts` para gráficos, `reactflow` para el grafo de vinculaciones.

## Calidad

`vitest` (lógica), `@playwright/test` (flujos críticos), `@sentry/nextjs` (errores),
`biome` (lint y formato en una sola herramienta, reemplaza ESLint + Prettier).

**Las pruebas de aislamiento RLS no son opcionales**: por cada tabla, un test que confirme
que la organización A no ve datos de la organización B.

## Lo que NO se instala

| Descartado | Por qué |
|---|---|
| Redux, Zustand o similar "por si acaso" | Con Server Components la mayor parte del estado es del servidor. Se añade solo si aparece un caso real. |
| NextAuth / Auth.js sobre Supabase | Dos modelos de sesión compitiendo. |
| Moment.js | Obsoleto y pesado. |
| Neo4j u otra base de grafos | `ADR-0001`. |
| Motores de billing extranjeros | `ADR-0002`. |
| Librerías de "admin panel" generadas | El costo de salirse de ellas supera lo que ahorran. |

---

# Skills, plugins y MCP de Claude Code

Conviene decirlo sin adornos: **no existe hoy un plugin de marketplace para "Next.js +
Supabase multi-tenant SaaS"**. El apalancamiento real viene de cuatro fuentes, y la cuarta
es la que responde a "que no se gasten tokens redescubriendo lo ya definido".

## 1. Servidores MCP ya conectados

| MCP | Para qué |
|---|---|
| **Supabase** | Inspeccionar esquema, aplicar migraciones, leer logs y *advisors* de seguridad sin salir del editor. Es el que más rinde en este stack. |
| **Vercel** | Despliegues, logs de ejecución y errores en producción. |
| **Linear / GitHub** | Issues y pull requests. |

Vale la pena además instalar las skills oficiales de Supabase, que traen guía de desarrollo
y de seguridad: `npx skills add supabase/agent-skills`.

## 2. Plugin `engineering` (ya habilitado)

| Skill | Cuándo usarla |
|---|---|
| `/engineering:architecture` | Escribir los próximos ADR. Los de este repo salieron de ahí. |
| `/engineering:code-review` | **Obligatorio** antes de fusionar código de RLS, cuotas o cobro. |
| `/engineering:testing-strategy` | Definir el plan de pruebas del módulo de screening. |
| `/engineering:debug` | Depuración estructurada cuando algo falla en producción. |
| `/engineering:documentation` | READMEs, runbooks y documentación de la API pública. |
| `/engineering:deploy-checklist` | Antes de cada release con migraciones de por medio. |

## 3. Plugin `pm-skills` (ya habilitado en este repo)

`/user-story-writing` para las historias, según `04-historias/README.md`.

## 4. Skills propias del proyecto — el ahorro real

Aquí está la respuesta directa a la preocupación de gastar tokens redefiniendo lo ya
definido. Cada patrón que se repite en el código merece una skill propia, **con el código de
referencia adentro**, para que el asistente no lo reinvente cada vez ni pregunte de nuevo:

| Skill a crear | Qué encapsula |
|---|---|
| `nueva-tabla-con-rls` | Migración, política RLS y prueba de aislamiento, como un solo patrón indivisible |
| `nueva-consulta-de-screening` | Verificar cuota, llamar al proveedor, persistir evidencia inmutable, registrar costo, descontar consumo |
| `nuevo-formulario` | Zod → React Hook Form → Server Action → manejo de errores y estados |
| `nuevo-reporte-exportable` | Estructura de datos, plantilla y generación en servidor |
| `revision-de-cumplimiento` | Lista de verificación antes de tocar código de auditoría, cuotas o cobro |

Se crean con la skill `skill-creator`. **Escribirlas es la inversión de mayor retorno del
proyecto** después del propio modelo de datos: cada una convierte una decisión ya tomada en
algo que no se vuelve a discutir.

## 5. `CLAUDE.md` por repositorio

El de specs ya existe. Cuando se creen los repos de backend y frontend, cada uno lleva el
suyo con sus convenciones, sus patrones y —sobre todo— **lo que nunca se hace**.

## Regla de oro

**El código de RLS, cuotas, cobro y evidencia de auditoría se revisa a mano, línea por
línea.** Es exactamente donde el código generado por IA parece correcto y no lo está, y
donde el error no se manifiesta como una excepción sino como una fuga de datos o una
factura mal calculada.
