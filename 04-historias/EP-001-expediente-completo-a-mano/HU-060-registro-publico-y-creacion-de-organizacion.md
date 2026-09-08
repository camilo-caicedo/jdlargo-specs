---
id: HU-060
titulo: Registro público y creación de organización
estado: en-revision
epica: EP-001
prioridad: Must
actualizado: 2026-09-07
---

# HU-060 — Registro público y creación de organización

> **Origen.** No es una historia del descubrimiento original: `PA-038` (resuelta 2026-09-05)
> decía "alta manual al principio, autoservicio después" y así quedó documentado en
> `ADR-0002` §2c, `02-producto/roadmap.md` y `04-historias/EP-007-capa-comercial/`. El
> **2026-09-07, Camilo decide adelantar el autoservicio al MVP actual** — decisión de negocio
> tomada directamente por él, no una pregunta abierta nueva. Esta historia construye el
> **mecanismo** de registro y creación de organización; lo que sigue sin existir es la capa
> comercial completa que lo rodea (selección de plan, cobro, verificación KYB de la empresa
> contratante) — eso sigue en `EP-007`, todavía bloqueado por `PA-043` (precios).

## Historia

**Como** Visitante sin cuenta
**quiero** crear mi propia cuenta y mi propia organización cliente sin depender de que alguien
me invite ni de que un operador la cree por mí
**para** empezar a usar la plataforma de inmediato.

## Contexto

Hasta esta historia, la única forma de tener una cuenta era el alta manual de sistema
(`HU-001`) o una invitación a una organización ya existente (`HU-056`). El mecanismo técnico
para "crear una organización con su primer administrador" **ya existe**:
`create_organization_with_admin` (`HU-001`, expuesto como `createOrganizationWithAdmin` en
`organizations/use-cases.ts`) — hoy solo se invoca desde tests y desde procesos manuales de
sistema. Esta historia es, en gran parte, ponerle una pantalla pública encima a algo que el
backend ya sabe hacer, más la creación de la cuenta de Supabase Auth de quien se registra.

A diferencia de `HU-056` (donde un Administrador ya decidió confiar en un correo concreto al
invitarlo), aquí **cualquiera** puede escribir cualquier correo en el formulario — incluido el
de otra persona sin su consentimiento. Por eso el correo se confirma antes de que la cuenta
tenga acceso pleno, a diferencia de la invitación, que no lo exige.

## Criterios de aceptación

```gherkin
Escenario: Registrar una cuenta nueva con su propia organización
  Dado un visitante sin cuenta
  Cuando se registra con un correo, una contraseña y un nombre de organización cliente
  Entonces queda creada su cuenta
  Y queda creada una organización cliente nueva con ese nombre
  Y queda como Administrador de esa organización cliente
  Y la creación queda registrada en la bitácora
```

```gherkin
Escenario: El correo debe confirmarse antes de dar acceso pleno
  Dado alguien que se registró con un correo que no controla
  Cuando intenta iniciar sesión sin haber confirmado el correo
  Entonces no obtiene acceso a ninguna organización cliente
  Y ve un mensaje indicando que revise su correo para confirmar la cuenta
```

```gherkin
Escenario: No se puede registrar dos veces el mismo correo
  Dado un correo que ya tiene una cuenta en la plataforma
  Cuando alguien intenta registrarse con ese mismo correo
  Entonces la operación es rechazada
  Y se le indica que ya existe una cuenta con ese correo y se le enlaza a iniciar sesión
```

```gherkin
Escenario: Dos registros independientes no comparten nada entre sí
  Dado que una persona se registra creando "Organización Uno"
  Y otra persona distinta se registra creando "Organización Dos"
  Entonces existen dos organizaciones clientes independientes
  Y cada una ve únicamente lo suyo, con el mismo aislamiento que el resto del sistema
```

```gherkin
Escenario: No se puede repetir el identificador único de organización
  Dado que ya existe una organización cliente con el identificador "hex-avis"
  Cuando alguien intenta registrarse creando una organización con ese mismo identificador
  Entonces la operación es rechazada
  Y se le indica que ese identificador ya está en uso y que elija otro

Escenario: Dos organizaciones pueden compartir nombre si el identificador es distinto
  Dado que ya existe una organización cliente llamada "Hexavis S.A.S."
  Cuando alguien se registra creando otra organización también llamada "Hexavis S.A.S." pero con
    un identificador distinto
  Entonces la operación se completa sin error
```

## Reglas de negocio

- Quien se registra queda como **Administrador** de la organización cliente que crea — mismo
  comportamiento que ya tiene `create_organization_with_admin` para el alta manual.
- El correo debe confirmarse (flujo estándar de Supabase Auth) antes de que la cuenta tenga
  acceso a cualquier organización cliente — a diferencia de `HU-056`, donde el administrador
  que invita ya asume esa responsabilidad por la persona invitada.
- **Esta historia no asigna ningún plan, cupo ni verifica la empresa (KYB).** La organización
  nace exactamente igual que una creada por alta manual hoy — sin ningún campo de plan, porque
  ese modelo (`HU-046`) todavía no existe y sigue bloqueado por `PA-043`. No se inventa un
  estado "de prueba" que no está en el modelo de datos.
- El **nombre** de la organización cliente no tiene que ser único en la plataforma (mismo
  criterio que ya rige hoy para el alta manual — dos organizaciones distintas pueden coincidir
  en nombre).
- Quien se registra también captura un **identificador único (`slug`)** para su organización,
  pensado para usarse en URLs más adelante — este sí es único en toda la plataforma. Se sugiere
  automáticamente a partir del nombre, pero la persona puede editarlo antes de enviar. Decisión
  de Camilo (2026-09-07): mejor que forzar unicidad de nombre, porque muchas empresas pueden
  llamarse parecido o igual, pero el identificador siempre resuelve sin ambigüedad.

## Fuera de alcance

- Selección de plan, cobro y facturación → `EP-007` (`HU-046` en adelante), sigue bloqueado
  por `PA-043`.
- Verificación KYB de la empresa contratante → `EP-007`, futuro.
- Invitar a otras personas desde el registro mismo → ya existe, `HU-056`, se usa después de
  tener organización.
- Recuperar contraseña → `HU-057`.
- Cualquier límite de cuántas organizaciones puede crear una misma persona — no pedido, no se
  inventa.
- Usar `organization.slug` en las rutas de la aplicación (`/app/[slug]` en vez de
  `/app/[organizationId]`) — esta historia solo captura y garantiza la unicidad del dato; el
  cambio de ruteo, si se hace, es trabajo aparte.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| Correo de registro | Sí | Formato válido; único en la plataforma (`user.email`, `HU-001`) | Sí (dato personal) |
| Contraseña | Sí | Política por defecto de Supabase Auth, mismo criterio que `HU-057` | Sí |
| `organization.name` | Sí | Texto no vacío (`HU-001`); no exige unicidad | No |
| `organization.slug` | Sí | Minúsculas, números y guiones; único en toda la plataforma | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Decisiones: `ADR-0002` §2c (actualizada 2026-09-07 — el mecanismo se adelanta al MVP; la
  capa comercial que lo rodea sigue igual de bloqueada)
- Requisito funcional: `RF-060`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna nueva. `PA-038` sigue resuelta; lo que cambió es cuándo se
  construye su mitad de mecanismo, no la respuesta en sí.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-001` (`createOrganizationWithAdmin` ya existe), `HU-055` (a dónde entra
  alguien recién registrado).
- **Habilita a:** adquisición de organizaciones clientes nuevas sin intervención manual de
  sistema — el propio `EP-007` ya advertía "si se abre registro público antes, hay que
  adelantarla" (refiriéndose a la capa comercial); esa capa **no** se adelanta con esta
  historia, solo el mecanismo de alta. Queda una brecha deliberada: organizaciones que se
  registran solas hoy no tienen plan ni cupo hasta que `HU-046` exista.
- **Riesgo:** el mismo que ya identificó `HU-056` para el correo — si alguien registra el
  correo de un tercero sin su consentimiento, la confirmación por correo es la única barrera.
  No omitirla nunca, ni siquiera por comodidad de producto.
