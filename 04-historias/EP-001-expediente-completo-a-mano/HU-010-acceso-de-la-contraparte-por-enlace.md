---
id: HU-010
titulo: Acceso de la contraparte por enlace
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-010 — Acceso de la contraparte por enlace

## Historia

**Como** Contraparte
**quiero** entrar a entregar mi información con un enlace, sin crear usuario ni contraseña
**para** poder completar lo que me piden sin fricción, viendo únicamente mi propio expediente.

## Contexto

La Fase 4 del documento del cliente introduce algo que no estaba en `ADR-0001`: **la contraparte
no tiene cuenta**. Entra con un enlace y un token, y aun así carga documentos de identidad y
datos financieros. Es una superficie pública y hay que tratarla como tal.

`08-desarrollo/arquitectura-de-aplicacion.md` fija las reglas de esa segunda superficie: el
token vale para **un único expediente**, tiene expiración configurable, es revocable, y **cada
uso queda registrado con fecha, hora, dirección de red y dispositivo**. El aislamiento sigue
siendo el mismo mecanismo de `HU-002`, pero lo que se propaga no es un usuario sino un
expediente. Es un segundo modo, no una excepción.

El segundo factor —un código de un solo uso— se contempla "cuando el riesgo lo justifique", y
por correo o por mensaje de texto. Lo segundo tiene un costo por mensaje que hoy no está
presupuestado (`PA-019`).

## Criterios de aceptación

```gherkin
Escenario: Entrar con un enlace vigente
  Dado un expediente enviado con su enlace de acceso vigente
  Cuando la contraparte abre el enlace
  Entonces accede a su expediente sin crear usuario ni contraseña
  Y el acceso queda registrado con fecha, hora, dirección de red y dispositivo
  Y el expediente transita a "en diligenciamiento" si estaba en "enviada"
```

```gherkin
Escenario: El enlace da acceso a un solo expediente
  Dado una contraparte que entró con el enlace del expediente A
  Cuando intenta acceder a cualquier dato del expediente B, aunque sea de la misma organización cliente
  Entonces la lectura no devuelve ninguna fila
  Y la escritura es rechazada por la política de la base de datos
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Un enlace expirado no deja entrar
  Dado un enlace de acceso cuya fecha de expiración ya pasó
  Cuando la contraparte lo abre
  Entonces no obtiene acceso al expediente
  Y ve un mensaje que le indica a quién dirigirse
  Y el intento queda registrado con fecha, hora y dirección de red
```

```gherkin
Escenario: Revocar un enlace corta el acceso de inmediato
  Dado un enlace de acceso vigente y en uso
  Cuando el responsable interno lo revoca
  Entonces el siguiente uso de ese enlace es rechazado
  Y la revocación queda registrada con quién la ejecutó y cuándo
  Y lo que la contraparte ya había aportado se conserva intacto
```

```gherkin
Escenario: Emitir un enlace nuevo invalida el anterior
  Dado un expediente con un enlace de acceso vigente
  Cuando el responsable interno emite un enlace nuevo para ese expediente
  Entonces el enlace anterior deja de dar acceso
  Y el expediente conserva un solo enlace vigente a la vez
  Y ambos hechos quedan registrados en la bitácora
```

```gherkin
Escenario: Segundo factor cuando la configuración lo exige
  Dado un expediente cuya configuración exige un factor adicional
  Cuando la contraparte abre el enlace
  Entonces se le solicita un código de un solo uso antes de mostrarle el expediente
  Y el acceso solo se concede tras verificar el código
  Y queda registrado que el acceso se hizo con segundo factor
```

```gherkin
Escenario: Límite de peticiones sobre la superficie expuesta
  Dado un número de peticiones al portal de la contraparte por encima del límite configurado
  Cuando se reciben peticiones adicionales desde el mismo origen
  Entonces se rechazan sin llegar a la capa de datos
  Y el hecho queda registrado
```

## Reglas de negocio

- El token de acceso vale para **un único expediente**. No existe un token que abarque varios,
  ni uno que sobreviva al cierre del expediente.
- Un expediente tiene **un solo enlace vigente a la vez**. Emitir uno nuevo invalida el anterior.
- El token tiene expiración, es revocable, y **cada uso se registra** con fecha, hora, dirección
  de red y dispositivo (Fase 4).
- El contexto que se propaga a la base de datos es el **expediente**, no un usuario. El
  aislamiento se apoya en el mismo mecanismo de `HU-002`, con tipo de actor `contraparte`.
- La ruta del portal vive bajo un segmento propio con su propio control de entrada, y **nunca
  comparte capa de acceso con la aplicación interna**.
- El límite de peticiones sobre el portal es estricto: es el punto expuesto del sistema.
- El segundo factor es configurable por la organización cliente. Por correo se resuelve con el
  proveedor ya elegido; por mensaje de texto implica costo no presupuestado (`PA-019`).
- La contraparte **no es un usuario del sistema**: no tiene rol, no aparece en la matriz de
  permisos y no puede ver nada fuera de su expediente.

## Fuera de alcance

- Qué ve y qué llena la contraparte dentro del expediente → `HU-011`, `HU-012` y `HU-013`.
- Los recordatorios automáticos antes de que expire el enlace (§46) → Fase 6, con los trabajos
  programados.
- La firma electrónica → Fase 2.
- La entrega del enlace, mientras `PA-031` no se responda: aquí se emite y se registra; quién lo
  hace llegar y por qué medio queda pendiente.
- Autenticación reforzada por biometría o por documento.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `acceso.organization_id` | Sí | Organización cliente existente | No |
| `acceso.expediente_id` | Sí | Un token pertenece a un solo expediente | No |
| `acceso.token` | Sí | Aleatorio, de longitud suficiente; se almacena su huella, no el valor | Sí |
| `acceso.expira_en` | Sí | Momento futuro; la duración sale de la configuración | No |
| `acceso.estado` | Sí | `vigente` \| `expirado` \| `revocado` \| `reemplazado` | No |
| `acceso.exige_segundo_factor` | Sí | Verdadero o falso; sale de la configuración | No |
| `uso_acceso.ocurrido_en` | Sí | Momento del uso | No |
| `uso_acceso.direccion_red` | Sí | Dirección desde la que se accedió | Sí (dato personal indirecto) |
| `uso_acceso.dispositivo` | Sí | Agente declarado por el navegador | Sí (dato personal indirecto) |
| `uso_acceso.resultado` | Sí | `concedido` \| `rechazado` con su causa | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 3 (generación del enlace), Fase 4, §33
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (portal de la contraparte),
  `ADR-0001` (aislamiento)
- Contexto: `00-contexto/actores-y-roles.md`, nota de diseño

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-028` — bloqueante para el comportamiento al expirar: si el
  expediente se cierra, si se emite un enlace nuevo solo, o si queda esperando al responsable
  interno. `PA-031` — quién entrega el enlace. `PA-019` — si el segundo factor por mensaje de
  texto es necesario o basta el correo. **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-008` (existe un expediente), `HU-009` (la entrada dispara una transición),
  `HU-002` (el aislamiento del que este es un segundo modo), `HU-006`.
- **Habilita a:** `HU-011`, `HU-012`, `HU-013`.
- **Riesgo:** es la única superficie del sistema abierta a internet sin usuario detrás. Un fallo
  aquí no expone una organización cliente: expone documentos de identidad de personas concretas.
