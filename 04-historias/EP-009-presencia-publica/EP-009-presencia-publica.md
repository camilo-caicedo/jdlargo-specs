---
id: EP-009
titulo: Presencia pública
capacidad: CAP-09
estado: borrador
actualizado: 2026-09-07
---

# EP-009 — Presencia pública

> **Origen.** No es una capacidad del descubrimiento original — no aparece en
> `02-producto/mapa-de-capacidades.md` ni en la escalera de MVPs del cliente. Se agrega el
> 2026-09-07 porque el producto, aunque ya tiene motor de cumplimiento y portal de contraparte
> (`EP-001`), no tiene ninguna puerta pública que explique qué es antes de que alguien tenga
> credenciales. Mismo patrón que `HU-055`–`HU-057`: vacío detectado, no PA sin responder.

## Objetivo

Dar una única página pública que explique qué es la plataforma y cómo entrar, sin exigir
cuenta ni exponer nada del dominio de cumplimiento.

## Por qué existe

`PA-038` ya cerró que el MVP **no tiene registro público** — el alta de organización cliente
es manual. Eso significa que esta página **no vende autoservicio**: es una carta de
presentación y una puerta de entrada para quien ya tiene o va a recibir credenciales, no un
embudo de conversión con formulario de registro. Es deliberadamente pequeña.

## Alcance

**Incluye:**

- Una página de inicio pública (`HU-058`) con encabezado (marca, navegación mínima, enlace a
  inicio de sesión), una sección que explica el producto, y pie de página.

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Registro público / autoservicio | No aplica al MVP | `PA-038`: alta manual |
| Otras páginas de marketing (precios, casos de uso, blog) | Después, si hace falta | No pedidas; esta épica es una sola página |
| CMS o edición de contenido sin desplegar código | Después, si hace falta | El contenido es estático por ahora |
| Analítica de marketing (pixel, GA, etc.) | Después, si se pide | No pedida, no se inventa |

## Actores involucrados

- **Visitante** — persona sin cuenta que llega a la plataforma desde un enlace o buscador.
- **Usuario** — quien ya tiene cuenta y usa el enlace de inicio de sesión de esta página como
  puerta de entrada (`HU-055`).

## Criterios de éxito

1. Existe una página pública que no exige cuenta ni depende de ninguna otra pantalla para
   cargar.
2. El contenido nunca usa "certificación"/"certificado" como si el producto certificara
   (`ADR-0006`) — es la única épica de cara al público, así que es donde ese principio más
   importa cumplirse.
3. No comparte capa de acceso ni layout con la app interna ni con el portal de la contraparte
   (`HU-010`) — es una tercera superficie, la más simple de las tres.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| [`HU-058`](HU-058-pagina-de-inicio-publica.md) | Página de inicio pública | Should | en-revision |
| [`HU-059`](HU-059-actualizar-landing-con-funcionalidad-real.md) | Actualizar la página de inicio con funcionalidad real | Should | borrador |

`HU-059` no se construye ahora — depende de que `EP-003`, `EP-004` o `EP-005` tengan algo
real que mostrar. Queda escrita para que la idea de "vista previa honesta del producto" no se
improvise otra vez sin la palanca de `ADR-0006` ya puesta.

## Dependencias

- **Preguntas abiertas:** ninguna.
- **Otras épicas:** coordina con `HU-055` (`EP-001`) para el destino del enlace de inicio de
  sesión — no depende de que esté construida para arrancar.
- **Decisiones:** `ADR-0006` (el producto no certifica; rige todo el copy de esta página).

## Riesgo abierto

Ninguno técnico. El riesgo real es de negocio: que el copy se escriba antes de que el
posicionamiento del producto (nombre, propuesta de valor exacta) esté cerrado con Juan David.
`HU-058` lo señala explícitamente en vez de inventar la propuesta de valor final.
