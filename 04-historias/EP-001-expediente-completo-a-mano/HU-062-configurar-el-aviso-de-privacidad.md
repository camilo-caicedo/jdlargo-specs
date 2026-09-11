---
id: HU-062
titulo: Configurar el aviso de privacidad de la organización cliente
estado: implementado
epica: EP-001
prioridad: Must
actualizado: 2026-09-10
---

# HU-062 — Configurar el aviso de privacidad de la organización cliente

> **Origen.** Vacío detectado 2026-09-10 probando `HU-011` en producción: `addPrivacyNotice`
> (backend de `HU-011`) nunca tuvo una pantalla que lo llame — solo se invoca desde los tests.
> Sin esto, ninguna organización cliente real puede tener un aviso de privacidad configurado, y
> el portal de la contraparte (`HU-011`/`HU-012`) queda permanentemente bloqueado en "Aviso de
> privacidad pendiente". Es la misma clase de hallazgo que originó `HU-061`.

## Historia

**Como** Administrador de la organización cliente
**quiero** escribir y publicar el aviso de privacidad y tratamiento de datos que verá cada
contraparte
**para** poder abrir expedientes que de verdad lleguen al portal, sin depender de que alguien
lo inserte a mano en la base de datos.

## Contexto

`privacy_notices` está modelada igual que el resto de la configuración de `ADR-0004`: una fila
por `(organización, versión de configuración)`, editable solo mientras esa versión está en
`borrador` — el mismo candado que ya rige tipos de contraparte y matriz de requisitos, y por la
misma razón: una vez publicada, la versión queda congelada para que los expedientes que la
citan (y los consentimientos que guardan su copia exacta, `HU-011`) tengan una referencia
estable. Esta historia no relaja esa regla — construye la pantalla que falta para operarla.

No existe hoy ninguna pantalla de configuración en la app interna
(`src/app/app/[slug]/` solo tiene `expedientes` y `miembros`). Esta historia es la primera, y
se limita a lo que ya bloquea producción — no intenta cubrir de una vez tipos de contraparte,
matriz de requisitos ni roles (eso es una autogestión más amplia, ya prevista como `CAP-05` /
`EP-005`, y queda fuera de alcance aquí).

## Criterios de aceptación

```gherkin
Escenario: Crear el aviso de privacidad de la versión en borrador
  Dado un Administrador con una versión de configuración en estado "borrador"
  Cuando escribe el texto del aviso, sus finalidades, responsable, encargado y canales de derechos, y guarda
  Entonces el aviso queda asociado a esa versión de configuración
  Y se puede editar de nuevo mientras la versión siga en "borrador"

Escenario: El aviso no se puede editar después de publicar
  Dado una versión de configuración ya publicada con su aviso de privacidad
  Cuando el Administrador intenta modificarlo
  Entonces la pantalla no permite editarlo directamente
  Y le ofrece crear una nueva versión en borrador (heredando el aviso vigente) para modificarlo ahí

Escenario: Publicar una versión exige que el aviso de privacidad exista
  Dado una versión en borrador sin aviso de privacidad configurado
  Cuando el Administrador intenta publicarla
  Entonces la publicación es rechazada indicando que falta el aviso de privacidad
  Y la versión sigue en borrador

Escenario: Sin permiso no hay acceso a la pantalla
  Dado un usuario sin el permiso "configuration:administer" sobre la organización
  Cuando intenta acceder a la configuración del aviso de privacidad
  Entonces no ve la opción en la navegación
  Y el acceso directo por URL es rechazado
```

## Reglas de negocio

- Solo se edita sobre una versión de configuración en `borrador` — mismo candado que ya aplica
  `addPrivacyNotice` (no se toca ese contrato).
- Si la organización no tiene ninguna versión en borrador, la pantalla ofrece crear una nueva a
  partir de la activa (reutiliza `createDraftConfiguration`, que ya hereda el aviso vigente —
  construido en la ronda de auditoría de `HU-011`).
- Publicar una versión sin aviso de privacidad configurado queda bloqueado — el punto entero de
  esta historia es que eso nunca vuelva a pasar en silencio.
- Las finalidades (`purposes`) se editan como una lista de `clave + descripción + si exige
  autorización` — mismo esquema que ya valida `privacyNoticePurposeSchema`, no una lista
  distinta.

## Fuera de alcance

- Configurar tipos de contraparte, matriz de requisitos o roles/permisos desde la UI — son
  historias propias, más grandes, y viven naturalmente en `CAP-05`/`EP-005` cuando se
  construya la autogestión completa. Esta historia no las anticipa ni las bloquea.
- Traducir el aviso a otro idioma.
- Una plantilla legal sugerida **dentro de esta pantalla** por estándar (SARLAFT/PTEE) — el
  editor no le propone texto normativo al Administrador mientras escribe; sigue siendo
  responsabilidad legal, no de producto.

> **Actualización 2026-09-10 (decisión de Camilo).** Distinto de lo anterior:
> `seedBaseConfiguration` ahora sí siembra un aviso de privacidad **genérico de arranque**
> (texto y finalidades placeholder) como parte de la versión 1 de toda organización nueva —
> mismo espíritu que el bootstrap de tipos de contraparte de `HU-007` — para que ninguna
> organización empiece sin aviso configurado y el bloqueo que originó esta historia
> (`HU-011` sin aviso publicable) no se repita por defecto. Esto **no** reemplaza el propósito
> de esta pantalla: el aviso sembrado es un punto de partida editable, no un texto listo para
> producción — el Administrador sigue teniendo que revisarlo y reemplazarlo con el suyo antes
> de operar con contrapartes reales. La pantalla de esta historia es precisamente el mecanismo
> para hacerlo.

## Datos y validaciones

Sin datos ni validaciones nuevas — reutiliza el contrato ya existente de
`addPrivacyNotice`/`getPrivacyNoticeForVersion` (`src/server/configuration/privacy-notice.ts`).
Esta historia es exclusivamente la interfaz que falta.

## Trazabilidad

- Épica: `EP-001` (vacío detectado construyendo `HU-011`, no una historia de `CAP-05` —
  ver Contexto).
- Capacidad: `CAP-01`
- Decisiones: `ADR-0004` (configuración versionada — borrador/publicado)

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-011` (el contrato de `privacy_notices` ya existe), `HU-004`
  (publicación de versiones de configuración, `configuration:administer`).
- **Habilita a:** que cualquier organización cliente real pueda usar `HU-011`/`HU-012` en
  producción sin intervención manual sobre la base de datos.
- **Riesgo:** ninguna otra entidad de configuración (tipos de contraparte, matriz, roles) tiene
  pantalla propia todavía — el mismo vacío existe ahí, solo que hoy no bloquea nada en
  producción porque esos datos se siembran una sola vez al crear la organización
  (`seedBaseConfiguration`). Cuando un cliente real necesite ajustar su matriz sin pasar por
  soporte, hace falta la misma clase de historia para esas entidades — no está cubierta aquí.
