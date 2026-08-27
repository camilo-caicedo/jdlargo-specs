---
id: HU-045
titulo: Recordatorios a la contraparte
estado: borrador
epica: EP-006
prioridad: Could
actualizado: 2026-08-27
---

# HU-045 — Recordatorios a la contraparte

## Historia

**Como** Usuario operativo
**quiero** que la plataforma le recuerde sola a la contraparte lo que le falta antes de que se le
venza el enlace
**para** dejar de perseguir documentos por correo, que es exactamente lo que vinimos a eliminar.

## Contexto

El §46 lo plantea como caso borde: *"la contraparte no completa el proceso a tiempo: recordatorios
automáticos antes de que expire el enlace; qué pasa si expira sin completarse"*. La primera mitad
es esta historia; la segunda es `PA-028`, todavía abierta.

`08-desarrollo/arquitectura-de-aplicacion.md` la ubica entre los trabajos programados, junto con
vencimientos y re-consultas.

Y conviene recordar por qué esto importa más de lo que parece: el modelo comercial del producto se
resume en *"para que tu equipo de cumplimiento dedique su tiempo a analizar riesgos y tomar
decisiones, **no a perseguir documentos**"* (§39). Esta historia es esa frase, implementada.

## Criterios de aceptación

```gherkin
Escenario: Recordatorio antes de que expire el enlace
  Dado un expediente en diligenciamiento con su enlace de acceso próximo a expirar
  Cuando faltan los días configurados
  Entonces se envía un recordatorio a la contraparte indicando qué le falta
  Y el envío queda registrado con su momento y su destinatario
```

```gherkin
Escenario: El recordatorio dice qué falta
  Dado un expediente con campos y documentos pendientes
  Cuando se genera el recordatorio
  Entonces indica qué requisitos siguen sin cubrir
  Y no incluye información del expediente más allá de lo necesario para completarlo
```

```gherkin
Escenario: No se recuerda lo que ya está completo
  Dado un expediente cuya contraparte ya entregó todo lo exigido
  Cuando corre el trabajo de recordatorios
  Entonces no se le envía ningún recordatorio
```

```gherkin
Escenario: La cadencia es configuración
  Dado dos organizaciones clientes con cadencias de recordatorio distintas
  Cuando se evalúan los mismos expedientes pendientes
  Entonces cada una envía según su configuración
  Y existe un número máximo de recordatorios por expediente
```

```gherkin
Escenario: También se avisa al responsable interno
  Dado un expediente cuyo enlace está por expirar sin completarse
  Cuando se envía el recordatorio a la contraparte
  Entonces el responsable interno queda informado del estado
  Y puede emitir un enlace nuevo si corresponde
```

```gherkin
Escenario: Recordatorio de renovación
  Dado un expediente vinculado con su actualización periódica próxima
  Cuando llega el momento configurado
  Entonces se le recuerda a la contraparte lo que debe actualizar
  Y el recordatorio distingue una renovación de una vinculación nueva
```

```gherkin
Escenario: El recordatorio no expone datos de más
  Dado un recordatorio enviado por correo
  Cuando se revisa su contenido
  Entonces no incluye datos personales de la contraparte más allá de lo imprescindible
  Y el acceso al expediente sigue exigiendo el enlace y, si aplica, el segundo factor
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los recordatorios
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta los recordatorios enviados con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Los recordatorios se envían **antes** de que expire el enlace de acceso, con la antelación y la
  cadencia que configure la organización cliente.
- Existe un **máximo de recordatorios** por expediente. Insistir sin límite es hostigar, y además
  perjudica la reputación del remitente de correo.
- El recordatorio indica **qué falta**, con el mínimo de datos personales necesario. El contenido
  del expediente vive detrás del enlace, no dentro del correo.
- No se recuerda lo ya completado.
- El responsable interno queda informado, para que pueda actuar si la contraparte no responde.
- Todo envío queda registrado con su momento y su destinatario.
- El trabajo es programado, idempotente y reintentable: un reintento no envía dos veces el mismo
  recordatorio.

## Fuera de alcance

- Qué ocurre si el enlace expira sin completarse → `PA-028`. Esta historia recuerda; no decide qué
  pasa después.
- Los recordatorios por mensaje de texto, mientras `PA-019` no se resuelva.
- Plantillas de correo personalizables por el cliente.
- La medición de tasas de apertura o de respuesta.
- Los avisos internos al equipo de cumplimiento → `HU-041` y `HU-043`.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `recordatorio.organization_id` | Sí | Organización cliente existente | No |
| `recordatorio.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `recordatorio.motivo` | Sí | `diligenciamiento_pendiente` \| `enlace_por_expirar` \| `renovacion` | No |
| `recordatorio.destinatario` | Sí | Correo de la contraparte registrado en el expediente | Sí (dato personal) |
| `recordatorio.enviado_en` | Sí | Momento; se escribe una sola vez | No |
| `recordatorio.numero_envio` | Sí | Menor o igual al máximo configurado | No |
| `configuracion.antelacion_dias` y `cadencia` | Sí | Por organización cliente | No |
| `configuracion.maximo_recordatorios` | Sí | Numérico, mayor que cero | No |

## Trazabilidad

- Épica: `EP-006`
- Capacidad: `CAP-06`
- Documento del cliente: §46, §39, Fase 21
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (trabajos programados)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-031` — bloqueante: si la plataforma envía correos a la contraparte o
  el enlace lo entrega el usuario operativo por su cuenta. Si es lo segundo, esta historia no
  existe tal como está escrita. `PA-028` — qué pasa al expirar. `PA-019` — mensajes de texto.
  **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-010` (enlace y su expiración), `HU-041` (plazos), `HU-042` (renovaciones).
- **Habilita a:** que el equipo de cumplimiento deje de perseguir documentos, que es la promesa
  comercial del producto (§39).
- **Riesgo:** un correo automático mal calibrado se convierte en correo no deseado y arrastra
  consigo la entrega de todos los demás avisos de la plataforma. El máximo de recordatorios no es
  una cortesía: protege el canal.
