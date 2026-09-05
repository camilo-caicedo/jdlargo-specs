---
id: ADR-0010
titulo: Aceptación electrónica primero; firma digital certificada como módulo opcional
estado: propuesto
fecha: 2026-09-05
reemplaza-a: —
reemplazado-por: —
---

# ADR-0010 — Firma y evidencia electrónica

## Contexto

`PA-020` preguntaba qué proveedor de firma digital certificada se usaría y desde qué fase. La
respuesta del cliente dejó dos cosas: una gestión en curso —Juan David está averiguando con un
proveedor si puede emitir certificados para que las contrapartes firmen la debida diligencia
desde la plataforma, y si el OTP por correo tiene respaldo legal— y una recomendación de no
hacer del MVP rehén de esa respuesta.

La Ley 527 de 1999 `(por validar)` distingue tres cosas que se confunden a diario: **mensaje
de datos**, **firma electrónica** y **firma digital** (la certificada, con entidad de
certificación). No todas las obligaciones exigen la tercera, y diseñar como si sí encarece el
producto y ata el calendario a un proveedor externo.

Relacionado: `PA-019` cerró que para el MVP basta **correo + enlace seguro con OTP por
correo**, y que el SMS es opcional para casos de mayor riesgo — lo que elimina un proveedor y
un costo por mensaje que no estaban presupuestados.

## Opciones consideradas

### Opción A — Firma digital certificada como requisito del MVP
- A favor: máximo valor probatorio desde el primer expediente.
- En contra: mete al calendario una dependencia de un tercero que todavía no está contratado
  ni cotizado; encarece cada firma; y no está claro que el caso de uso lo exija.

### Opción B — Aceptación y evidencia electrónica ahora, firma certificada como módulo
- A favor: el MVP avanza sin bloqueo; cubre la mayoría de los casos; la firma certificada se
  enchufa después sin rehacer el flujo.
- En contra: hay que revisar jurídicamente el valor probatorio de lo que sí se implementa, y
  algún cliente puede exigir la certificada antes de lo previsto.

### Opción C — Elegir ya un proveedor y diseñar alrededor de él
- A favor: una sola forma de firmar, sin abstracciones.
- En contra: acopla el producto a un proveedor antes de saber qué necesita cada cliente. La
  Ley 527 no impone una modalidad única; el producto tampoco debería.

## Decisión

**Opción B.**

1. **El MVP implementa aceptación y evidencia electrónica**, no firma certificada: enlace
   firmado con expiración, OTP por correo, registro de aceptación con fecha, hora, medio,
   dirección de origen y la versión exacta del documento aceptado. **Revisado jurídicamente
   antes de salir a producción** — esto no se da por bueno solo porque sea razonable.
2. **La firma digital certificada es un módulo opcional**, detrás de un **adaptador de firma**
   con proveedor configurable por organización cliente. El expediente guarda la evidencia y el
   documento firmado, sea cual sea la modalidad.
3. **El segundo factor por defecto es el correo** (`PA-019`). SMS y WhatsApp quedan como
   proveedor opcional, activable por política o por nivel de riesgo. Ahorra costo variable sin
   tocar el flujo básico.
4. **La modalidad se elige por caso, no por producto.** Un mensaje de datos, una firma
   electrónica y una firma digital son tres niveles distintos, y la matriz de requisitos puede
   exigir uno u otro por tipo de contraparte y por documento.

## Consecuencias

- **Positivas:** el MVP no depende de una negociación con un proveedor de certificación; el
  costo por firma en el caso común es cercano a cero; cuando llegue el cliente que exija firma
  certificada, se conecta un adaptador en vez de rehacer el módulo.
- **Negativas / costo asumido:** el valor probatorio de la aceptación electrónica es menor que
  el de una firma certificada, y esa diferencia hay que explicarla en el contrato en vez de
  esconderla. Si la revisión jurídica concluye que el caso de uso exige firma digital, esta
  decisión se cae y hay que adelantar el módulo.
- **Qué habría que revisar si el contexto cambia:** las dos respuestas que Juan David está
  buscando (`PA-041`). Cualquiera de las dos puede reabrir esto.

## Trazabilidad

- Cierra `PA-019` y `PA-020`. Deriva `PA-041`.
- Afecta: `HU-010`, `HU-011`, `HU-022`, `HU-045`, `08-desarrollo/arquitectura-de-aplicacion.md`.
