---
id: HU-021
titulo: Vigencias y estados del documento
estado: borrador
epica: EP-002
prioridad: Must
actualizado: 2026-08-27
---

# HU-021 — Vigencias y estados del documento

## Historia

**Como** Analista de Cumplimiento
**quiero** que cada documento tenga su vigencia y su estado, y que el sistema sepa cuándo dejó
de servir
**para** no descubrir en una auditoría que el expediente se aprobó con un certificado que ya
estaba vencido cuando se cargó.

## Contexto

La Fase 7 del documento del cliente define el ciclo completo:
`No recibido → Recibido → En validación → Válido → Requiere revisión → Rechazado → Vencido`.
La Fase 1 (`HU-013`) construyó todo salvo el último estado, porque las vigencias dependen de
saber leer fechas del documento, que es lo que aporta la extracción.

La §34 advierte contra un error frecuente: **no asumir que todos los documentos tienen la misma
vigencia**. Un certificado de existencia y representación legal y un documento de identidad no
caducan igual, y esa diferencia es configuración del cliente, no una constante del programa
(`ADR-0004`).

Hay un matiz de procedencia que conviene fijar: la fecha de vencimiento **declarada** por la
contraparte y la **extraída** del documento pueden no coincidir. Ahí no se elige la más cómoda:
se abre una discrepancia como cualquier otra (`HU-019`).

## Criterios de aceptación

```gherkin
Escenario: La vigencia sale de la configuración, no del programa
  Dado una versión de configuración que define la vigencia de cada tipo documental
  Cuando se carga un documento de un tipo con vigencia definida
  Entonces el sistema calcula su fecha de caducidad a partir de esa configuración y de la fecha de expedición
  Y dos tipos documentales distintos pueden tener vigencias distintas
```

```gherkin
Escenario: Un documento vence y cambia de estado
  Dado un documento válido cuya fecha de caducidad ya pasó
  Cuando se evalúan las vigencias
  Entonces el documento transita al estado vencido
  Y se genera una alerta sobre el expediente al que pertenece
  Y la transición queda registrada con el momento y el motivo
```

```gherkin
Escenario: Un documento vencido no cubre su requisito
  Dado un expediente con un tipo documental obligatorio cuyo único documento está vencido
  Cuando se consulta qué requisitos están cubiertos
  Entonces ese requisito aparece como no cubierto
  Y el expediente no puede avanzar a decisión por ese motivo
```

```gherkin
Escenario: Cargar un documento ya vencido se rechaza al momento
  Dado un tipo documental con vigencia definida
  Cuando la contraparte carga un documento cuya fecha de expedición lo deja vencido desde el primer día
  Entonces se le indica que el documento no está vigente
  Y el documento queda en estado requiere revisión, no válido
```

```gherkin
Escenario: Fechas declarada y extraída que no coinciden
  Dado un documento cuya fecha de vencimiento declarada por la contraparte difiere de la extraída del propio documento
  Cuando se calcula la conciliación
  Entonces se abre una discrepancia sobre esa fecha
  Y el sistema no elige ninguna de las dos por su cuenta
  Y la vigencia queda pendiente hasta que la discrepancia se resuelva
```

```gherkin
Escenario: Reemplazar un documento vencido conserva el anterior
  Dado un documento vencido en un expediente
  Cuando la contraparte carga una versión nueva y vigente del mismo tipo documental
  Entonces la versión nueva pasa a cubrir el requisito
  Y la versión vencida se conserva completa, con su huella digital y su historia
  Y el requisito vuelve a aparecer como cubierto
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las vigencias
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta documentos próximos a vencer con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
  Y no obtiene ninguno de "Beta Ficticia S.A.S."
```

## Reglas de negocio

- La vigencia de cada tipo documental es **configuración versionada** (`ADR-0004`), no una
  constante. Puede expresarse como duración desde la expedición, como fecha propia del
  documento, o como "sin vencimiento".
- El estado del documento sigue el ciclo completo de la Fase 7, incluido `vencido`, y solo
  cambia por transiciones válidas.
- Un documento vencido **no cubre** su requisito. El expediente lo refleja de inmediato.
- Las fechas de expedición y vencimiento son afirmaciones con origen. Si la declarada y la
  extraída difieren, se abre discrepancia (`HU-019`) y la vigencia queda pendiente.
- Un documento nunca se borra al vencer ni al reemplazarse: se conserva con su huella digital
  (`HU-013`).
- El vencimiento genera una **alerta**, no una decisión. Qué hacer con ella es del analista.

## Fuera de alcance

- El trabajo programado que revisa vigencias a diario y avisa antes de que venzan → Fase 6. Aquí
  el estado se calcula y se evalúa; la vigilancia continua es de la Fase 6.
- La renovación del expediente completo por vencimiento → Fase 6.
- La gestión de la alerta como caso → Fase 3, donde nace el concepto de caso.
- La verificación de la autenticidad del documento contra el emisor → Fase 3.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `tipo_documental.vigencia` | Sí | `sin_vencimiento` \| duración desde la expedición \| fecha propia del documento | No |
| `documento.fecha_expedicion` | Condicional | Obligatoria si la vigencia se calcula desde ella; es una afirmación con origen | Sí |
| `documento.fecha_caducidad` | Condicional | Calculada o declarada; es una afirmación con origen | Sí |
| `documento.estado` | Sí | `no_recibido` \| `recibido` \| `en_validacion` \| `valido` \| `requiere_revision` \| `rechazado` \| `vencido` | No |
| `transicion_documento.motivo` | Condicional | Obligatorio al rechazar o marcar para revisión | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: Fase 7, §34 (no todos los documentos tienen la misma vigencia)
- Decisiones: `ADR-0004` (la vigencia es configuración), `ADR-0005` (las fechas son afirmaciones)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-018` — qué tipos documentales hay que soportar y con qué vigencias
  en el primer cliente. `PA-027` — precedencia entre la fecha declarada y la extraída. **Queda en
  `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-013` (documentos), `HU-017` (fechas extraídas), `HU-019` (discrepancias),
  `HU-007` (la configuración de vigencias).
- **Habilita a:** el monitoreo de vencimientos de la Fase 6 y la renovación periódica.
