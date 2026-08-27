---
id: HU-022
titulo: Firma electrónica de niveles 1 y 2
estado: borrador
epica: EP-002
prioridad: Should
actualizado: 2026-08-27
---

# HU-022 — Firma electrónica de niveles 1 y 2

## Historia

**Como** Contraparte
**quiero** revisar el formulario ya completo, corregir lo que haga falta y firmarlo
**para** dejar constancia de que la información que entregué es la mía y la doy por buena.

## Contexto

Es la Fase 19 del documento del cliente, y su primera línea es una corrección a la versión
anterior que conviene no perder: **no es cierto que "dirección de red + fecha y hora + huella
digital" equivalga a una firma legalmente válida en cualquier circunstancia**. Lo que se exige
es un método que permita identificar a quien firma y demuestre su aprobación, siendo confiable y
apropiado para el propósito.

De ahí los tres niveles configurables por el cliente:

| Nivel | Qué es | Quién lo construye |
|---|---|---|
| 1 | Aceptación electrónica con evidencia: huella del documento, dirección de red, fecha y hora, versión aceptada | Propio |
| 2 | Firma reforzada: el nivel 1 más un factor adicional verificado | Propio |
| 3 | Firma digital certificada | **Proveedor externo acreditado** (`PA-020`), fuera del MVP |

La capa de firma vive detrás de un puerto único, para que el nivel 3 se enchufe después sin
tocar el flujo del expediente. Y la §17 sitúa la firma **antes** de la decisión: la contraparte
revisa el formulario autollenado, corrige y firma; solo entonces el expediente llega completo al
Oficial de Cumplimiento.

## Criterios de aceptación

```gherkin
Escenario: Revisar el formulario autollenado antes de firmar
  Dado un expediente con datos declarados y extraídos ya conciliados
  Cuando la contraparte entra con su enlace de acceso a revisar
  Entonces ve el formulario con los valores vigentes y de dónde viene cada uno
  Y puede corregir los que no correspondan
  Y cada corrección se registra como afirmación nueva atribuida a ella
```

```gherkin
Escenario: Firmar con nivel 1
  Dado una organización cliente configurada con firma de nivel 1
  Y una contraparte que revisó su formulario
  Cuando firma
  Entonces se registra la evidencia con la huella digital del contenido firmado, la dirección de red, la fecha, la hora y la versión aceptada
  Y el expediente transita al estado siguiente
  Y la firma queda registrada en la bitácora
```

```gherkin
Escenario: Firmar con nivel 2
  Dado una organización cliente configurada con firma de nivel 2
  Cuando la contraparte firma
  Entonces se le solicita un código de un solo uso antes de completar la firma
  Y la evidencia registra que hubo un factor adicional verificado, además de todo lo del nivel 1
  Y sin ese factor la firma no se completa
```

```gherkin
Escenario: La firma cubre un contenido concreto e inmutable
  Dado un expediente firmado
  Cuando se consulta la firma
  Entonces se obtiene la huella digital de exactamente lo que se firmó
  Y esa huella permite comprobar que el contenido no cambió después
  Y si el contenido cambiara, la firma queda señalada como no correspondiente
```

```gherkin
Escenario: Modificar lo firmado exige firmar de nuevo
  Dado un expediente ya firmado por la contraparte
  Cuando se registra una afirmación nueva sobre un campo cubierto por la firma
  Entonces la firma anterior se conserva con su huella y su momento
  Y el expediente exige una firma nueva antes de avanzar a decisión
```

```gherkin
Escenario: El nivel de firma es configuración, no código
  Dado dos organizaciones clientes, una configurada con nivel 1 y otra con nivel 2
  Cuando una contraparte firma en cada una
  Entonces cada firma exige lo que su configuración define
  Y la diferencia proviene de la configuración, no de una condición escrita en el programa
```

```gherkin
Escenario: El nivel 3 no está disponible todavía
  Dado una organización cliente que intenta configurar firma de nivel 3
  Cuando publica esa configuración
  Entonces el sistema indica que el nivel 3 requiere un proveedor externo no integrado
  Y no permite dejar expedientes bloqueados esperando una firma imposible
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre las firmas
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta firmas con su contexto de usuario propagado
  Entonces obtiene únicamente las de expedientes de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- El nivel de firma exigido es **configuración de la organización cliente** (`ADR-0004`), no una
  constante del programa.
- La firma de nivel 1 registra: huella digital del contenido firmado, dirección de red, fecha,
  hora, y la versión exacta de lo que se aceptó.
- La firma de nivel 2 añade un **factor adicional verificado**. Sin verificación, no hay firma.
- La firma cubre un contenido concreto. Si ese contenido cambia después, **la firma anterior no
  vale para el contenido nuevo**: se conserva, y hace falta firmar otra vez.
- La evidencia de firma es inmutable y de solo inserción.
- El nivel 3 se enchufa detrás del mismo puerto cuando exista proveedor (`PA-020`). **No entra en
  el MVP**, y la plataforma no debe permitir configurarlo de forma que bloquee expedientes.
- La plataforma **no afirma que un nivel de firma sea suficiente** para un propósito legal
  determinado. Ofrece niveles y evidencia; la elección es del cliente y su asesoría (§40).

## Fuera de alcance

- La firma digital certificada de nivel 3 y su proveedor → `PA-020`.
- La firma de documentos por parte de los usuarios internos del cliente.
- El sellado de tiempo por una autoridad de certificación.
- La validación jurídica de la suficiencia de cada nivel: es del cliente, no del producto.
- El código de un solo uso por mensaje de texto, mientras `PA-019` no se resuelva; por correo se
  cubre con el proveedor ya elegido.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `firma.organization_id` | Sí | Organización cliente existente | No |
| `firma.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `firma.nivel` | Sí | `1` \| `2`; el `3` no está disponible | No |
| `firma.huella_contenido` | Sí | Huella digital de lo firmado; se calcula al firmar | No |
| `firma.direccion_red` | Sí | Dirección desde la que se firmó | Sí (dato personal indirecto) |
| `firma.firmada_en` | Sí | Momento; se escribe una sola vez | No |
| `firma.factor_adicional` | Condicional | Obligatorio si el nivel es 2; registra el medio y su verificación | Sí |
| `firma.version_contenido` | Sí | Versión exacta del contenido firmado | No |
| `firma.estado` | Sí | `vigente` \| `no_correspondiente` (si el contenido cambió después) | No |

## Trazabilidad

- Épica: `EP-002`
- Capacidad: `CAP-02`
- Documento del cliente: Fase 19, Fase 17, §40
- Decisiones: `08-desarrollo/arquitectura-de-aplicacion.md` (puerto único de firma, tres niveles)
- Normativa: efectos jurídicos de los mensajes de datos `(por validar — Ley 527 de 1999, ver
  SUP-008)`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-019` (segundo factor por mensaje de texto), `PA-020` (proveedor de
  firma certificada y desde qué fase). **Queda en `borrador`.**
- **Supuestos:** `SUP-008` (las referencias normativas no están verificadas; la plataforma no
  afirma suficiencia jurídica de ningún nivel).
- **Depende de:** `HU-010` (la contraparte entra), `HU-019` (revisa lo conciliado), `HU-005`,
  `HU-009`.
- **Habilita a:** la decisión (`HU-015`) sobre un expediente firmado, que es como la §17 lo
  prevé.
- **Riesgo:** prometer que el nivel 1 sirve "para cualquier circunstancia" es exactamente el
  error que el cliente corrigió en su propio documento. Conviene que ni la interfaz ni el
  material comercial lo insinúen.
