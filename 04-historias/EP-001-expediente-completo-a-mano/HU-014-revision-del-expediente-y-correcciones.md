---
id: HU-014
titulo: Revisión del expediente y solicitud de correcciones
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-014 — Revisión del expediente y solicitud de correcciones

## Historia

**Como** Analista de Cumplimiento
**quiero** revisar lo que entregó la contraparte y poder devolverle lo que está mal indicando
qué corregir
**para** que el expediente llegue completo a la decisión y no se apruebe algo que nadie miró.

## Contexto

Entre que la contraparte termina de entregar y el Oficial de Cumplimiento decide hay un paso que
el documento del cliente da por supuesto y que conviene escribir: **alguien mira lo recibido**.
En la Fase 1 esa revisión es enteramente humana, porque todavía no existen ni la extracción con
IA (Fase 2), ni la verificación externa (Fase 3), ni el motor de riesgo (Fase 4).

Es también el punto donde aterriza el caso borde de la §46: *"documento ilegible o rechazado:
reintento, sin perder lo ya diligenciado en otras fases"*. Devolver un documento no puede
costarle a la contraparte volver a llenar el formulario.

La revisión **no decide**. Marca documentos, pide correcciones y deja el expediente listo para
quien sí decide (`HU-015`). Es la separación de funciones de la §30 aplicada al recorrido.

## Criterios de aceptación

```gherkin
Escenario: Ver el expediente completo para revisarlo
  Dado un expediente en estado "documentos recibidos"
  Cuando el Analista de Cumplimiento lo abre
  Entonces ve lo que la contraparte declaró, con su origen y su momento
  Y ve cada documento entregado, con su tipo documental, su huella digital y quién lo cargó
  Y ve qué requisitos de la matriz están cubiertos y cuáles no
```

```gherkin
Escenario: Marcar un documento como válido
  Dado un documento en estado recibido
  Cuando el Analista de Cumplimiento lo marca como válido
  Entonces el documento queda en estado válido
  Y queda registrado quién lo validó y cuándo
  Y el hecho aparece en la bitácora del expediente
```

```gherkin
Escenario: Rechazar un documento sin perder lo demás
  Dado un documento ilegible entregado por la contraparte
  Cuando el Analista de Cumplimiento lo rechaza indicando el motivo
  Entonces el documento queda en estado rechazado con ese motivo
  Y el expediente vuelve a permitir a la contraparte cargar ese tipo documental
  Y lo que la contraparte ya había diligenciado en el formulario se conserva intacto
  Y el resto de documentos conserva su estado
```

```gherkin
Escenario: Solicitar corrección devuelve el expediente a la contraparte
  Dado un expediente en revisión con observaciones pendientes
  Cuando el Analista de Cumplimiento solicita correcciones
  Entonces el expediente transita al estado de diligenciamiento
  Y la contraparte ve qué se le pide corregir y por qué
  Y el enlace de acceso sigue siendo válido o se emite uno nuevo, según lo que resuelva PA-028
  Y la solicitud queda registrada en la bitácora
```

```gherkin
Escenario: La revisión no decide
  Dado un usuario con rol de Analista de Cumplimiento en una organización cliente cuya política no le otorga el permiso de decidir
  Cuando intenta aprobar o rechazar la vinculación
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Dar el expediente por listo para decisión
  Dado un expediente cuyos requisitos obligatorios están cubiertos y cuyos documentos están validados
  Cuando el Analista de Cumplimiento lo da por revisado
  Entonces el expediente transita a "pendiente de decisión"
  Y queda registrado quién lo revisó y cuándo
```

```gherkin
Escenario: Un expediente incompleto no pasa a decisión
  Dado un expediente con un requisito obligatorio sin cubrir
  Cuando el Analista de Cumplimiento intenta darlo por revisado
  Entonces la operación es rechazada indicando qué falta
  Y el expediente permanece en revisión
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la revisión
  Dado un analista miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta expedientes pendientes de revisión con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y un intento de marcar un documento de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- Revisar es marcar y observar; **revisar no es decidir** (§30). Quién puede decidir lo determina
  la matriz de permisos de la organización cliente (`HU-003`).
- Marcar un documento deja registro de quién lo validó o lo rechazó, cuándo y con qué motivo. Un
  rechazo sin motivo no se admite.
- Rechazar un documento **no invalida el resto del expediente** y no borra lo diligenciado
  (§46).
- Solicitar correcciones es una **transición** del expediente (`HU-009`), con motivo obligatorio.
- El sistema calcula qué requisitos de la matriz están cubiertos y cuáles no; es un cálculo
  sobre las afirmaciones y los documentos, no un campo que alguien marque a mano.
- Dar un expediente por revisado exige que sus requisitos obligatorios estén cubiertos.
  `PA-029` define si el Oficial de Cumplimiento puede saltarse esa condición dejando constancia.
- La revisión no crea afirmaciones sobre la contraparte: no convierte lo declarado en verificado.
  Verificar exige una fuente externa (Fase 3) y es un origen distinto (§2).

## Fuera de alcance

- La decisión y su fundamento → `HU-015`.
- La comprobación automática de lo declarado contra el documento → Fase 2.
- La verificación contra fuentes externas y el screening → Fase 3.
- Las alertas y su gestión como casos → Fase 3.
- La asignación de expedientes a analistas, colas de trabajo y balanceo de carga.
- La conversación con la contraparte dentro de la plataforma: aquí se le indica qué corregir, no
  se abre un canal de mensajería.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `revision.organization_id` | Sí | Organización cliente existente | No |
| `revision.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `revision.revisor_id` | Sí | Usuario con permiso de revisar | No |
| `revision.ocurrida_en` | Sí | Momento; se escribe una sola vez | No |
| `observacion.documento_id` | Condicional | Obligatorio si la observación es sobre un documento | No |
| `observacion.requisito` | Condicional | Obligatorio si la observación es sobre un campo | No |
| `observacion.motivo` | Sí | Texto no vacío | No |
| `documento.estado` | Sí | Solo transiciones válidas del estado del documento (`HU-013`) | No |
| `documento.validado_por` | Sí | Usuario que marcó el estado | No |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 7, Fase 15, §30, §46
- Decisiones: `ADR-0005` (revisar no cambia el origen de una afirmación)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-028` — si al solicitar correcciones el enlace de acceso sigue
  sirviendo o hay que emitir uno nuevo. `PA-029` — si el expediente puede pasar a decisión con
  requisitos pendientes. **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-012` (hay datos que revisar), `HU-013` (hay documentos que revisar),
  `HU-009`, `HU-003`, `HU-006`.
- **Habilita a:** `HU-015`.
