---
id: HU-011
titulo: Aviso de privacidad y evidencia del consentimiento
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-011 — Aviso de privacidad y evidencia del consentimiento

## Historia

**Como** Contraparte
**quiero** saber antes de entregar nada qué datos míos se van a tratar, para qué, quién
responde por ellos y cómo ejerzo mis derechos
**para** decidir con información si acepto, y que quede constancia exacta de qué versión acepté
y cuándo.

## Contexto

Es el punto que más cambió frente a la versión anterior del documento del cliente, y conviene
citarlo tal cual: *"no todo tratamiento de datos necesita el mismo tipo de autorización, y un
casilla marcada no vuelve lícito por sí solo cualquier tratamiento"* (§5).

De ahí que lo configurable no sea solo el texto sino la **base jurídica**: qué se trata, con qué
finalidad, si hace falta autorización explícita y en qué casos, quién es responsable y quién
encargado, qué derechos tiene el titular y por qué canal los ejerce.

Lo que la plataforma garantiza es la **evidencia**: qué versión exacta del aviso se mostró, qué
aceptó la persona, cuándo, por qué medio y desde dónde. La plataforma no dictamina si esa base
jurídica es suficiente; eso es un concepto jurídico y el producto no los emite (§34).

Si la contraparte no acepta lo que jurídicamente sea necesario, la solicitud queda **rechazada
por la contraparte** y el proceso termina ahí para esa persona (§5).

## Criterios de aceptación

```gherkin
Escenario: Mostrar el aviso antes de pedir cualquier dato
  Dado una contraparte que entra por su enlace de acceso por primera vez
  Cuando accede al expediente
  Entonces se le muestra el aviso de privacidad de la versión de configuración citada por el expediente
  Y no se le solicita ningún dato ni documento antes de resolver el aviso
```

```gherkin
Escenario: Registrar la evidencia de la aceptación
  Dado un aviso de privacidad mostrado a la contraparte
  Cuando la contraparte acepta
  Entonces queda registrada la aceptación con la versión exacta del aviso, la fecha, la hora, el medio y la dirección de red
  Y el expediente puede avanzar a la entrega de información
  Y la aceptación queda registrada en la bitácora
```

```gherkin
Escenario: La contraparte no acepta
  Dado un aviso de privacidad que exige autorización explícita
  Cuando la contraparte no acepta
  Entonces el expediente transita a "rechazada por la contraparte"
  Y se notifica al responsable interno
  Y no se le solicita ningún dato ni documento
  Y queda registrado qué versión del aviso rechazó y cuándo
```

```gherkin
Escenario: Cambiar el aviso no altera lo ya aceptado
  Dado una contraparte que aceptó la versión 2 del aviso de privacidad
  Cuando la organización cliente publica una versión 3 del aviso
  Entonces la evidencia sigue diciendo que aceptó la versión 2
  Y el texto de la versión 2 sigue siendo legible tal como se mostró
```

```gherkin
Escenario: La configuración decide si hace falta autorización explícita
  Dado una finalidad de tratamiento que la organización cliente configuró como amparada por otra base jurídica
  Cuando la contraparte accede al expediente
  Entonces se le informa el tratamiento sin solicitarle autorización explícita para esa finalidad
  Y queda registrada la base jurídica declarada y la versión del aviso mostrado
```

```gherkin
Escenario: La evidencia sobrevive al cierre del expediente
  Dado un expediente cerrado
  Cuando se consulta la evidencia del consentimiento
  Entonces se obtiene la versión aceptada, la fecha, la hora, el medio y la dirección de red
  Y ningún camino del sistema permite modificarla ni borrarla
```

## Reglas de negocio

- El aviso de privacidad y la base jurídica son **contenido de una versión de configuración**
  (`HU-004`): se publican, no se editan. Un aviso mostrado tiene que seguir siendo legible años
  después, exactamente como se mostró.
- La evidencia registra siempre: versión del aviso, qué se aceptó, fecha, hora, medio y dirección
  de red. Es de solo inserción, como todo lo que sostiene una auditoría.
- **Mostrar un aviso no es obtener un consentimiento.** Son dos hechos distintos y se registran
  por separado.
- Cuando la configuración exige autorización explícita y la contraparte no la da, el expediente
  termina en "rechazada por la contraparte" y **no se le pide ningún dato**.
- La plataforma no evalúa si la base jurídica configurada es suficiente. Registra qué declaró el
  cliente y qué aceptó el titular; el juicio jurídico es del cliente, que es el responsable del
  tratamiento (§40).
- El aviso debe poder informar, cuando aplique, que hay tratamiento por parte de terceros fuera
  del país (§33). Qué proveedores se declaran depende de `PA-021`.

## Fuera de alcance

- La redacción del aviso: la escribe el cliente con su asesoría jurídica. La plataforma lo
  almacena versionado y lo muestra.
- Los canales para ejercer derechos del titular —consulta, rectificación, supresión— como
  funcionalidad de la plataforma. Aquí se informan; atenderlos es del cliente.
- La firma electrónica del formulario → Fase 2. Aceptar el aviso y firmar el expediente son
  hechos distintos.
- La revocación posterior del consentimiento por parte del titular y sus efectos sobre un
  expediente ya decidido.
- Las notificaciones al responsable interno, mientras `PA-031` no se responda.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `aviso.version_configuracion_id` | Sí | Versión publicada (`HU-004`) | No |
| `aviso.texto` | Sí | Texto no vacío; inmutable una vez publicado | No |
| `aviso.finalidades` | Sí | Al menos una finalidad declarada | No |
| `aviso.exige_autorizacion` | Sí | Verdadero o falso, por finalidad | No |
| `aviso.responsable` y `aviso.encargado` | Sí | Texto no vacío | No |
| `aviso.canales_derechos` | Sí | Texto no vacío | No |
| `consentimiento.organization_id` | Sí | Organización cliente existente | No |
| `consentimiento.expediente_id` | Sí | Expediente al que pertenece | No |
| `consentimiento.aviso_version` | Sí | Versión exacta mostrada | No |
| `consentimiento.resultado` | Sí | `aceptado` \| `no_aceptado` | No |
| `consentimiento.ocurrido_en` | Sí | Momento; se escribe una sola vez | No |
| `consentimiento.medio` | Sí | Por dónde se aceptó | No |
| `consentimiento.direccion_red` | Sí | Dirección desde la que se aceptó | Sí (dato personal indirecto) |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: §5 (Fase 5), §33, §34, §40, §18
- Decisiones: `ADR-0004` (el aviso es configuración versionada), `ADR-0005` (la evidencia es un
  evento inmutable)
- Normativa: tratamiento de datos personales en Colombia `(por validar — Ley 1581 de 2012 y su
  alcance exacto, ver SUP-008)`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-031` — cómo se notifica al responsable interno cuando la
  contraparte no acepta. `PA-021` — qué proveedores de IA se declaran en el aviso, dado que
  implican transferencia internacional de datos (§33). **Queda en `borrador`.**
- **Supuestos:** `SUP-004` (los datos pueden alojarse en Estados Unidos, confirmado), `SUP-008`
  (las referencias normativas no están verificadas: el aviso se almacena y se muestra, la
  plataforma no afirma su suficiencia jurídica).
- **Depende de:** `HU-004` (versión que contiene el aviso), `HU-010` (la contraparte entró),
  `HU-009` (la no aceptación es una transición), `HU-006`.
- **Habilita a:** `HU-012` y `HU-013`. Sin resolver el aviso no se pide ningún dato.
- **Riesgo:** es la historia con más superficie legal y menos superficie técnica. La tentación es
  resolverla con una casilla; el documento del cliente advierte expresamente que eso no basta.
