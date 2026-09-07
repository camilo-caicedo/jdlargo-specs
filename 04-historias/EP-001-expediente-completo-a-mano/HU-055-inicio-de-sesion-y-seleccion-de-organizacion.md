---
id: HU-055
titulo: Inicio de sesión y selección de organización
estado: en-revision
epica: EP-001
prioridad: Must
actualizado: 2026-09-07
---

# HU-055 — Inicio de sesión y selección de organización

> **Origen.** No es una historia del descubrimiento original: se detectó como vacío al auditar
> el avance de la Fase 1. `HU-001` modeló identidad, membresía y roles, y excluyó
> expresamente de su alcance "pantallas: la Fase 0 no entrega interfaz" y el flujo de
> autenticación en sí. Ninguna historia posterior lo recogió, y sin embargo `HU-008` en
> adelante ya asume un usuario autenticado con contexto de organización propagado. Se ubica en
> `EP-001` (no en `EP-000`) porque es la primera pantalla de la app interna, y `EP-000` se
> define como la épica "verificable sin interfaz".

## Historia

**Como** Usuario con cuenta ya creada en la plataforma (`HU-001`)
**quiero** iniciar sesión con mi correo y contraseña, y elegir con cuál de mis organizaciones
clientes voy a trabajar si tengo más de una
**para** entrar a mis expedientes sin que nadie confunda el rol o los datos de una organización
cliente con los de otra.

## Contexto

`HU-001` dejó dicho el modelo (`Usuario › Membresía › Organización › Roles`) y una consecuencia
explícita: "la interfaz necesita selector de organización" — sin construirla. También dejó el
criterio de aceptación "autenticarse no otorga por sí solo acceso a ninguna organización
cliente": esta historia es la que hace ese principio tangible en pantalla, no solo en la base
de datos.

El modelo de contraseña se apoya en `@supabase/ssr` (ya instalado, primer uso real en
`08-desarrollo/librerias-y-entorno-ia.md`: "cubre todos los flujos de cuenta, incluido el
refresco de sesión"). Esta historia **no reinventa** sesión, refresco de token ni
almacenamiento de contraseña — los usa. El modelo de correo y contraseña (en vez de enlace
mágico) se elige porque `HU-057` (recuperar contraseña) solo tiene sentido si existe una
contraseña que recuperar; es una consecuencia de diseño, no una decisión aparte.

## Criterios de aceptación

```gherkin
Escenario: Iniciar sesión con una sola organización cliente
  Dado un usuario con cuenta activa y una única membresía activa, en "Alfa Ficticia S.A.S."
  Cuando inicia sesión con su correo y contraseña correctos
  Entonces obtiene una sesión válida
  Y entra directamente con "Alfa Ficticia S.A.S." como organización activa, sin selector
```

```gherkin
Escenario: Iniciar sesión con varias organizaciones clientes
  Dado un usuario con membresías activas en "Alfa Ficticia S.A.S." y "Beta Ficticia S.A.S."
  Cuando inicia sesión con su correo y contraseña correctos
  Entonces ve un selector con únicamente esas dos organizaciones clientes
  Y al elegir una, esa queda como organización activa de la sesión
```

```gherkin
Escenario: Cambiar de organización activa sin cerrar sesión
  Dado un usuario con sesión activa y organización activa "Alfa Ficticia S.A.S."
  Y una segunda membresía activa en "Beta Ficticia S.A.S."
  Cuando pide cambiar de organización activa y elige "Beta Ficticia S.A.S."
  Entonces la organización activa pasa a ser "Beta Ficticia S.A.S."
  Y no vuelve a pedirle correo ni contraseña
```

```gherkin
Escenario: Un usuario no puede elegir una organización a la que no pertenece
  Dado un usuario con sesión activa y sin ninguna membresía en "Beta Ficticia S.A.S."
  Cuando intenta fijar "Beta Ficticia S.A.S." como organización activa
  Entonces la operación es rechazada
  Y su organización activa no cambia
```

```gherkin
Escenario: Iniciar sesión sin ninguna membresía activa
  Dado un usuario con cuenta activa y sin ninguna membresía activa
  Cuando inicia sesión con su correo y contraseña correctos
  Entonces obtiene una sesión válida
  Y ve un mensaje indicando que no tiene acceso a ninguna organización cliente
  Y no llega a ver ningún expediente ni dato de dominio
```

```gherkin
Escenario: Credenciales incorrectas
  Dado un usuario que intenta iniciar sesión con una contraseña incorrecta
  Cuando envía el formulario
  Entonces la operación es rechazada con un mensaje genérico
  Y ese mensaje no revela si el correo existe o no en la plataforma
```

```gherkin
Escenario: Cerrar sesión
  Dado un usuario con sesión activa
  Cuando cierra sesión
  Entonces la sesión deja de ser válida
  Y un intento posterior de ver un expediente exige iniciar sesión de nuevo
```

## Reglas de negocio

- El correo y la contraseña se validan contra la cuenta ya creada por `HU-001`; esta historia
  no crea cuentas nuevas (eso es `HU-056`, por invitación — no hay registro público, `PA-038`).
- La organización activa de una sesión **siempre** es una organización cliente en la que el
  usuario tiene una membresía `active` — nunca un valor que el cliente pueda fijar sin que el
  servidor lo verifique contra las membresías reales.
- Iniciar sesión otorga una sesión; no otorga por sí solo acceso a ningún dato de dominio de
  ninguna organización cliente (`HU-001`, ya probado ahí a nivel de datos; aquí se prueba a
  nivel de lo que la persona efectivamente puede ver).
- Un usuario sin ninguna membresía activa puede autenticarse, pero no tiene ninguna
  organización que elegir ni ningún expediente que ver.
- El mensaje de credenciales inválidas nunca distingue "el correo no existe" de "la contraseña
  es incorrecta" — evita que alguien use el formulario para averiguar qué correos están
  registrados.

## Fuera de alcance

- Registro público de organizaciones nuevas — no aplica al MVP (`PA-038`).
- Invitar a un nuevo miembro y que acepte → `HU-056`.
- Recuperar contraseña olvidada → `HU-057`.
- Segundo factor de autenticación para usuarios internos — `HU-001` ya lo dejó fuera; Supabase
  Auth lo trae incluido (`auth.mfa`, `librerias-y-entorno-ia.md`) para cuando exista una
  historia que lo pida.
- Pantalla de perfil (cambiar nombre, foto, correo, contraseña estando ya autenticado).
- Administrar roles y permisos con interfaz → `HU-038`, Fase 5.
- Cualquier diseño visual concreto de la pantalla — lo define el plan de arquitectura al
  construirla, no esta historia de negocio.

## Datos y validaciones

No agrega tablas nuevas: consulta `users` y `memberships`, ambas de `HU-001`, y se apoya en el
modelo de sesión de `@supabase/ssr`.

| Campo / concepto | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| Correo de inicio de sesión | Sí | Debe corresponder a un `user.email` existente | Sí (dato personal) |
| Organización activa de la sesión | Condicional | Obligatoria si hay al menos una membresía `active`; siempre una de ellas | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: §30 (roles), §31 (multiempresa) — mismos que `HU-001`, esta historia
  construye la pantalla que faltaba
- Decisiones: `ADR-0001` §7 (identidad global y membresía), `08-desarrollo/librerias-y-entorno-ia.md`
  (Supabase Auth para sesión y refresco)
- Requisito funcional: `RF-055`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio — reutiliza el modelo de `HU-001` sin extenderlo.
- **Depende de:** `HU-001` (usuarios y membresías), `HU-002` (aislamiento — la organización
  activa elegida aquí es el contexto que `withTenantContext` propaga después).
- **Habilita a:** `HU-056`, `HU-057`, y en la práctica cualquier pantalla interna futura: es el
  punto de entrada de toda la app que no sea el portal público de `HU-010`.
- **Riesgo:** si la organización activa se toma de un valor que el cliente controla sin
  verificarla contra las membresías reales, se rompe el aislamiento entre organizaciones desde
  la primera pantalla — el mismo riesgo que `HU-002` ya previene a nivel de base de datos, aquí
  aplicado a nivel de interfaz.
