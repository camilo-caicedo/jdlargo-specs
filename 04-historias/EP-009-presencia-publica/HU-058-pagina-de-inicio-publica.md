---
id: HU-058
titulo: Página de inicio pública
estado: en-revision
epica: EP-009
prioridad: Should
actualizado: 2026-09-07
---

# HU-058 — Página de inicio pública

> **Origen.** No es una historia del descubrimiento original: se agrega el 2026-09-07 a
> petición explícita, con el mismo patrón de `HU-055`–`HU-057` — vacío detectado, no una `PA-xxx`
> sin responder. Hoy la ruta raíz (`/`) de `jdlargo-app` es el `page.tsx` por defecto de
> `create-next-app`; no existe ninguna superficie pública que explique el producto.

## Historia

**Como** Visitante sin cuenta que llega a la plataforma desde un enlace o un buscador
**quiero** ver en la página de inicio qué hace el producto, de un vistazo, y cómo iniciar
sesión si ya tengo acceso
**para** decidir si es lo que busco sin tener que pedirle a nadie una demostración primero.

## Contexto

Esta es la **tercera superficie** de la aplicación, después de la app interna (autenticada,
`HU-055` en adelante) y el portal de la contraparte (sin cuenta pero con token acotado a un
expediente, `HU-010`). Esta es la más simple de las tres: **estática, sin dato de dominio,
sin sesión, sin token**. No comparte layout ni capa de acceso con ninguna de las otras dos.

El copy está sujeto a `ADR-0006`: el producto **no certifica** — nunca puede decir "empresa
certificada" ni presentar un resultado como una certificación. Esta historia no cierra la
propuesta de valor exacta ni el nombre final del producto (`AGENTS.md` los marca como
provisionales todavía) — el contenido textual concreto se redacta con lo que ya está resuelto
(`00-contexto/vision.md`, `ADR-0006`) y se marca `(por confirmar con Juan David)` donde haga
falta un mensaje de marketing que todavía no se ha validado con él, en vez de inventarlo.

## Criterios de aceptación

```gherkin
Escenario: Visitar la página de inicio sin sesión
  Dado un visitante sin cuenta ni sesión activa
  Cuando abre la ruta raíz de la aplicación
  Entonces ve un encabezado con la marca y navegación mínima
  Y ve una sección que explica qué hace el producto
  Y ve un pie de página
  Y no se le exige ninguna credencial para ver el contenido
```

```gherkin
Escenario: Entrar a iniciar sesión desde la página de inicio
  Dado un visitante en la página de inicio
  Cuando hace clic en el enlace o botón de "Iniciar sesión"
  Entonces llega a la pantalla de inicio de sesión de la aplicación
```

```gherkin
Escenario: La página se ve completa en un dispositivo móvil
  Dado un visitante en un dispositivo con pantalla angosta
  Cuando abre la página de inicio
  Entonces todo el contenido es legible sin desplazamiento horizontal
  Y el encabezado y el pie de página se adaptan al ancho disponible
```

```gherkin
Escenario: El contenido nunca promete certificación
  Dado el texto completo de la página de inicio
  Cuando se revisa contra `ADR-0006`
  Entonces ninguna frase usa "certifica", "certificado" o "certificación" como si el producto
    acreditara o garantizara un resultado
```

## Reglas de negocio

- Es una página **estática**: no lee ni escribe ninguna tabla de dominio, no abre sesión de
  base de datos con contexto de tenant.
- Nunca comparte layout, componentes de navegación ni capa de acceso con la app interna
  (`HU-055` en adelante) ni con el portal de la contraparte (`HU-010`) — son tres superficies
  independientes, cada una con su propia raíz de enrutador.
- El lenguaje sigue `ADR-0006` sin excepción: el producto consulta, verifica, evalúa,
  documenta y sugiere; nunca certifica.
- No incluye ningún formulario de captura de datos (ni de registro, ni de contacto) — `PA-038`
  sigue vigente: no hay autoservicio en el MVP. Si más adelante se quiere un formulario de
  contacto comercial, es una historia aparte, no una ampliación silenciosa de esta.

> **Decisión de enrutamiento (confirmada 2026-09-07).** La ruta raíz (`/`) es **siempre** esta
> página pública — nunca la app autenticada. `HU-055` (inicio de sesión), `HU-056`
> (aceptar invitación) y `HU-057` (recuperar contraseña) viven cada una bajo su propia ruta
> propia (p. ej. `/login`, `/invitaciones/[token]`, `/recuperar-contrasena`), y la app
> autenticada en sí vive bajo su propio prefijo (p. ej. `/app`) — nunca en `/`. Mismo principio
> que ya separa al portal de la contraparte bajo `/portal`: cada superficie tiene su propia
> raíz de enrutador, y `/` es la de esta historia.

## Fuera de alcance

- Registro público / autoservicio — `PA-038` sigue sin registro público en el MVP.
- Cualquier otra página de marketing (precios, casos de uso, blog, changelog) — esta historia
  es una sola página, no un sitio.
- CMS o cualquier forma de editar el contenido sin desplegar código.
- Internacionalización — solo español, mismo criterio que el resto del producto de cara al
  usuario final.
- Analítica de marketing (pixel de conversión, Google Analytics, etc.) — no pedida.
- La propuesta de valor y el nombre comercial definitivos — se usan los ya validados
  (`00-contexto/vision.md`, `ADR-0006`); lo que no esté validado se marca explícito, no se
  inventa.

## Datos y validaciones

Ninguno. No hay tabla de dominio ni persistencia asociada a esta historia.

## Trazabilidad

- Épica: `EP-009`
- Capacidad: `CAP-09`
- Decisiones: `ADR-0006` (posicionamiento sin certificación)
- Contexto: `00-contexto/vision.md`
- Requisito funcional: `RF-058`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio — el contenido se apoya en `vision.md`/`ADR-0006`, ya
  cerrados; lo que no lo esté, queda marcado `(por confirmar)` en el copy, no inventado.
- **Depende de:** nada técnicamente. Coordina con `HU-055` para el destino del enlace de
  inicio de sesión y para que la app autenticada no reclame la misma ruta raíz.
- **Habilita a:** nada aguas abajo — es la puerta de entrada, no un prerrequisito técnico de
  otra historia.
- **Riesgo:** de negocio, no técnico — que el copy se congele antes de que Juan David valide
  la propuesta de valor exacta. Mitigado dejando esos puntos marcados explícitos en vez de
  redactados como si fueran definitivos.
