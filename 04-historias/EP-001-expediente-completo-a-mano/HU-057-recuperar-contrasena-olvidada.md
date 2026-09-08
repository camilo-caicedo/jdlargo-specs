---
id: HU-057
titulo: Recuperar contraseña olvidada
estado: implementado
epica: EP-001
prioridad: Must
actualizado: 2026-09-07
---

# HU-057 — Recuperar contraseña olvidada

> **Origen.** Igual que `HU-055` y `HU-056`: vacío detectado al auditar la Fase 1. `HU-001`
> excluyó expresamente "recuperación de contraseña" de su alcance. Sin esta historia, un
> usuario que olvida su contraseña queda sin ninguna vía para volver a entrar salvo pedirle a
> alguien que le cree una cuenta nueva — lo que además violaría "identidad global" (`HU-001`).

## Historia

**Como** Usuario con cuenta en la plataforma que olvidó su contraseña
**quiero** pedir un enlace de restablecimiento a mi correo y fijar una contraseña nueva
**para** volver a entrar sin depender de que alguien más intervenga por mí.

## Contexto

`HU-055` construye el inicio de sesión con correo y contraseña; esta historia es su
contraparte obligatoria — un modelo de contraseña sin recuperación deja a cualquier persona que
la olvide fuera de la plataforma para siempre. Se apoya en el flujo de recuperación ya incluido
en Supabase Auth (`@supabase/ssr`, `librerias-y-entorno-ia.md`: "cubre todos los flujos de
cuenta, incluido... recuperación de cuenta"). No se construye verificación de identidad propia
más allá de lo que el proveedor ya ofrece — es exactamente el caso que ese catálogo de
librerías señala como "ya resuelto y probado", no una pieza de dominio de este producto.

## Criterios de aceptación

```gherkin
Escenario: Solicitar recuperación con un correo registrado
  Dado un usuario con cuenta cuyo correo es "usuario@ejemplo.com"
  Cuando pide recuperar su contraseña con ese correo
  Entonces recibe un correo con un enlace de restablecimiento vigente por tiempo limitado
  Y la solicitud queda registrada en la bitácora
```

```gherkin
Escenario: Solicitar recuperación con un correo que no existe
  Dado que "inexistente@ejemplo.com" no corresponde a ninguna cuenta
  Cuando alguien pide recuperar la contraseña de ese correo
  Entonces ve el mismo mensaje genérico que si el correo sí existiera
  Y no se envía ningún correo
```

```gherkin
Escenario: Establecer una contraseña nueva con un enlace vigente
  Dado un enlace de restablecimiento vigente y no usado
  Cuando el usuario lo abre y fija una contraseña nueva
  Entonces puede iniciar sesión con la contraseña nueva
  Y ya no puede iniciar sesión con la anterior
  Y el enlace queda inutilizado para cualquier uso posterior
```

```gherkin
Escenario: Un enlace de restablecimiento expirado o ya usado no funciona
  Dado un enlace de restablecimiento vencido, o uno que ya se usó una vez
  Cuando el usuario lo abre
  Entonces no puede fijar ninguna contraseña con ese enlace
  Y ve un mensaje que lo invita a solicitar uno nuevo
```

## Reglas de negocio

- El mensaje tras solicitar recuperación es **el mismo** exista o no una cuenta con ese correo
  — mismo principio de `HU-055` para no revelar qué correos están registrados.
- Un enlace de restablecimiento es de un solo uso y expira; el valor exacto de expiración lo
  fija la configuración por defecto de Supabase Auth, sin una regla adicional de la plataforma
  por encima.
- Fijar una contraseña nueva invalida la anterior de inmediato — no queda ninguna sesión ni
  contraseña previa utilizable.
- No se define en esta historia ninguna política de complejidad de contraseña más allá de la
  que Supabase Auth aplica por defecto. Si el negocio necesita una política propia (longitud
  mínima distinta, caracteres exigidos), es una historia aparte, no una suposición de esta.
- **Enrutamiento (confirmado 2026-09-07, junto con `HU-058`):** recuperar contraseña vive en
  su propia ruta (p. ej. `/recuperar-contrasena`), distinta de `/` (página pública, `HU-058`)
  y de la ruta de inicio de sesión (`HU-055`).

## Fuera de alcance

- Cambiar la contraseña estando ya autenticado (pantalla de perfil) — no pedida, no es
  "olvidé mi contraseña".
- Verificación de identidad adicional (pregunta de seguridad, documento, etc.) — no la exige
  ninguna fuente de este proyecto; se apoya solo en la posesión del correo, como el resto de
  proveedores de este tipo.
- Segundo factor de autenticación — mismo alcance que `HU-055`, sigue fuera.
- Política de complejidad de contraseña propia de la plataforma — ver Reglas de negocio.

## Datos y validaciones

No agrega tablas nuevas: usa el flujo de recuperación de Supabase Auth sobre `users` (`HU-001`).
Lo único que aporta este producto es la bitácora de la solicitud.

| Campo / concepto | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| Correo de la solicitud | Sí | Formato de correo válido; no se valida contra la existencia de la cuenta antes de responder | Sí (dato personal) |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Decisiones: `08-desarrollo/librerias-y-entorno-ia.md` (Supabase Auth para todo el flujo de
  cuenta, sin reconstruirlo)
- Requisito funcional: `RF-057`

## Dependencias y riesgos

- **Preguntas abiertas:** ninguna.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-055` (existe un modelo de correo y contraseña que recuperar).
- **Habilita a:** nada aguas abajo directamente; cierra el ciclo de autenticación que abre
  `HU-055`.
- **Riesgo:** ninguno propio de negocio — el riesgo real es de implementación (no reconstruir
  a mano lo que Supabase Auth ya ofrece), señalado en Contexto.
