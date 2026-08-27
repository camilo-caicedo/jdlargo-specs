---
id: EP-001
titulo: Un expediente completo, a mano
estado: borrador
capacidad: CAP-01
actualizado: 2026-08-27
---

# EP-001 — Un expediente completo, a mano

> **Es la primera fase que se puede poner en producción.** Un expediente sin IA, sin consulta a
> listas y sin motor de riesgo parece pobre, y aun así con solo esto el cliente ya reemplaza su
> hoja de cálculo y su cadena de correos, y empieza a tener trazabilidad real. Es la Fase 1 del
> `02-producto/roadmap.md`.

## Objetivo

Recorrer un expediente entero de punta a punta —crear la solicitud, dejar entrar a la
contraparte por su enlace, recibir lo que declara y los documentos que carga, revisarlo y
decidir— de modo que al final quede **cerrado y reconstruible**, sin que ninguna pieza del
proceso ocurra fuera de la plataforma.

## Por qué existe y por qué va primero

El roadmap lo dice sin adornos: es la fase que suele saltarse, y es donde se descubre si el
modelo de datos aguanta **antes** de haber construido seis módulos encima. Todo lo que viene
después —extracción, verificación, riesgo, monitoreo— se apoya en que exista un expediente que
ya funciona.

Deja contestadas seis de las dieciséis preguntas del criterio de aceptación del cliente (§44):

| # | Pregunta de la §44 | Historia que la contesta |
|---|---|---|
| A | Quién era la contraparte | `HU-008` |
| B | Qué información entregó | `HU-012` |
| C | Qué documentos presentó | `HU-013` |
| L | Quién tomó la decisión | `HU-015` |
| M | Por qué la tomó | `HU-015` |
| N | Qué condiciones quedaron | `HU-015` |

`HU-016` es la que hace que esas seis respuestas se puedan **mostrar juntas**, que es lo que
pide la §18.

## Alcance

**Incluye:**

- Tipos de contraparte y matriz de requisitos como contenido de la configuración, cargados a
  mano (`HU-007`).
- Solicitud de vinculación que abre un expediente sobre una contraparte (`HU-008`).
- Máquina de estados del expediente como datos, no como una columna de texto (`HU-009`).
- Acceso de la contraparte por enlace y token acotado a un solo expediente (`HU-010`).
- Aviso de privacidad, base jurídica y evidencia del consentimiento (`HU-011`).
- Formulario dinámico armado desde la matriz, cuyas respuestas se guardan como afirmaciones
  declaradas (`HU-012`).
- Carga de los documentos exigidos, con huella digital del archivo (`HU-013`).
- Revisión humana de lo recibido y solicitud de correcciones (`HU-014`).
- Decisión del Oficial de Cumplimiento, con fundamento, condiciones y vigencia (`HU-015`).
- Consulta del expediente completo y reconstruible (`HU-016`).

**No incluye —y por qué:**

| Fuera de alcance | Dónde va | Razón |
|---|---|---|
| Extracción con IA y conciliación entre lo declarado y lo extraído | Fase 2 | En la Fase 1 el único origen es `declarado`. La estructura ya lo admite (`HU-005`) |
| Estados y vigencias del documento (`No recibido → … → Vencido`) | Fase 2 | El roadmap los agrupa con la extracción. Aquí el documento se recibe y se revisa, no caduca |
| Firma electrónica de nivel 1 | Fase 2 | Decisión explícita del roadmap. La Fase 1 cierra el expediente sin firma de la contraparte |
| Verificación externa, consulta a listas, coincidencias y alertas | Fase 3 | Sin catálogo de fuentes no hay a quién preguntar |
| Personas relacionadas, beneficiario final y motor de riesgo | Fase 4 | El expediente de la Fase 1 es sobre la contraparte, no sobre su estructura |
| Debida diligencia intensificada | Fase 4 | Se activa por factores que aquí todavía no se calculan |
| Interfaz de administración de la configuración | Fase 5 | La matriz se carga a mano hasta entonces (`HU-007`) |
| Monitoreo continuo, renovaciones y recordatorios automáticos | Fase 6 | Un expediente decidido en la Fase 1 no vuelve a moverse solo |
| Reutilización de un sujeto ya conocido en otro expediente (§46) | Fase 4 | Requiere el motor de relaciones. Aquí cada expediente parte de cero |
| Exportación del expediente a PDF o Excel | Fase 6 | `HU-016` lo hace reconstruible en pantalla; el reporte exportable llega con el panel |

## Actores involucrados

- **Usuario operativo** — crea la solicitud desde su área.
- **Contraparte** — entra por el enlace, acepta el aviso, diligencia y carga documentos.
- **Analista de Cumplimiento** — revisa lo recibido y pide correcciones.
- **Oficial de Cumplimiento** — decide. Es el único que cierra el expediente.
- **Auditor / Consulta** — reconstruye el expediente sin poder tocarlo.
- **Sistema** — aplica la máquina de estados, arma el formulario y registra evidencia. Nunca decide.

## Criterios de éxito

1. **Un expediente real, de punta a punta, sin salir de la plataforma.** Ninguna parte del
   recorrido ocurre por correo o por hoja de cálculo.
2. **Las seis preguntas de la §44 se contestan desde el propio expediente**, sin construir un
   reporte aparte.
3. **Cero datos del expediente sin procedencia.** Todo lo que declara la contraparte queda como
   afirmación de origen `declarado` (`HU-005`).
4. **Cero decisiones sin fundamento.** No existe forma de cerrar un expediente sin persona,
   motivo y evidencia asociada.
5. **Cero transiciones inválidas.** Un intento de saltar un estado es un error de dominio, no
   una columna que alguien actualizó a mano.
6. **El expediente cita la versión de configuración con la que se armó**, y esa versión sigue
   siendo legible aunque después se publiquen otras.

## Historias

| ID | Historia | Prioridad | Estado |
|----|----------|-----------|--------|
| `HU-007` | Tipos de contraparte y matriz de requisitos | Must | borrador |
| `HU-008` | Crear la solicitud de vinculación y abrir el expediente | Must | borrador |
| `HU-009` | Máquina de estados del expediente | Must | borrador |
| `HU-010` | Acceso de la contraparte por enlace | Must | borrador |
| `HU-011` | Aviso de privacidad y evidencia del consentimiento | Must | borrador |
| `HU-012` | Formulario dinámico de identificación | Must | borrador |
| `HU-013` | Carga de los documentos exigidos | Must | borrador |
| `HU-014` | Revisión del expediente y solicitud de correcciones | Must | borrador |
| `HU-015` | Decisión del Oficial de Cumplimiento | Must | borrador |
| `HU-016` | Expediente electrónico reconstruible | Must | borrador |

Orden sugerido: `HU-007` → `HU-008` → `HU-009` → `HU-010` → `HU-011` → `HU-012` → `HU-013` →
`HU-014` → `HU-015` → `HU-016`. Es el orden del recorrido, y coincide con la §45: primero el
modelo de cumplimiento, la pantalla al final.

## Dependencias

- **Épicas:** `EP-000` entera. Ninguna historia de esta épica puede escribir una fila antes de
  que exista el aislamiento entre organizaciones, la bitácora y la estructura de afirmaciones.
- **Preguntas abiertas:** `PA-018` (cuántos estándares y tipos de contraparte), `PA-025`
  (cuándo congela el expediente su versión de configuración), `PA-028` (qué pasa si expira el
  enlace), `PA-029` (si se puede decidir con requisitos pendientes), `PA-030` (formatos y
  tamaño de documentos), `PA-031` (quién entrega el enlace a la contraparte), `PA-019` (si el
  segundo factor por SMS es necesario).
- **Supuestos:** `SUP-001`, `SUP-003` (contexto colombiano), `SUP-008` (las normas citadas por
  el cliente no están verificadas).
- **Decisiones:** `ADR-0004` (formulario dinámico y matriz como datos), `ADR-0005`
  (afirmaciones y decisiones como eventos), `08-desarrollo/arquitectura-de-aplicacion.md`
  (portal de la contraparte, máquina de estados, subida de archivos con URL firmada).

## Riesgo abierto

`PA-029` es la más incómoda de las seis: si el Oficial de Cumplimiento puede decidir con
requisitos pendientes, la máquina de estados necesita un camino de excepción registrada; si no
puede, necesita un bloqueo duro. Son dos diseños distintos de `HU-009` y `HU-015`, y elegir uno
por omisión es exactamente el tipo de supuesto silencioso que este repositorio no admite.
