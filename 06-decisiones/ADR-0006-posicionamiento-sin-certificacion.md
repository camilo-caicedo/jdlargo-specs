---
id: ADR-0006
titulo: El producto no certifica — automatiza y traza la debida diligencia
estado: propuesto
fecha: 2026-09-05
reemplaza-a: —
reemplazado-por: —
---

# ADR-0006 — Posicionamiento: automatización, no certificación

## Contexto

El proyecto nació llamándose "validación, verificación y **certificación** de entidades". El
documento funcional del cliente evita el término en todo su texto (§39), y al cerrar `PA-003`
y `PA-022` el cliente lo dijo sin rodeos:

> "No vamos a certificar, solo se va a consultar información y se va a sugerir."
>
> "Es una herramienta de apoyo, no certifica en ningún momento."

La palabra no es un detalle de marketing. "Certificar" implica que un tercero acreditado
respalda un estado de cumplimiento con una vigencia. La plataforma no acredita nada, no
responde por el resultado, y la decisión sigue siendo del Oficial de Cumplimiento del cliente.
Prometer certificación crea una **falsa garantía** frente a un supervisor y traslada al
proveedor una responsabilidad que no puede sostener.

## Opciones consideradas

### Opción A — Conservar "certificación" en el nombre y el discurso
- A favor: es el término que el mercado colombiano reconoce y busca; vende más rápido.
- En contra: es inexacto; expone al proveedor a reclamos si un cliente certificado resulta
  sancionado; el propio cliente lo rechazó; obliga a explicar en cada venta que la
  certificación no certifica.

### Opción B — Reposicionar como automatización y evidencia de debida diligencia
- A favor: describe lo que el producto hace de verdad; la propuesta de valor se vuelve más
  creíble, no menos; reduce riesgo comercial y regulatorio; alinea el nombre con el glosario y
  con `ADR-0005`.
- En contra: hay que reeducar al comprador que llegó buscando "certificación"; el nombre del
  proyecto cambia.

### Opción C — "Certificación" solo como nombre de un reporte
- A favor: conserva algo de la familiaridad del término.
- En contra: es la peor de las tres. Un documento llamado "certificado" será leído como
  garantía por quien lo reciba, aunque el contrato diga otra cosa.

## Decisión

**Opción B.** Se retira "certificación" del posicionamiento principal del producto.

1. **Nombre y discurso comercial:** "Automatización de Debida Diligencia" /
   *Compliance Due Diligence Platform*. El producto se posiciona alrededor de cinco palabras:
   automatización, evidencia, screening, riesgo y monitoreo.
2. **Salidas del sistema** (cierra la parte de producto de `PA-003` y `PA-007`):
   - **Informe de Debida Diligencia** — PDF y JSON, con fecha, metodología, fuentes
     consultadas, resultados, alertas, decisiones y versión normativa aplicada.
   - **Constancia de proceso ejecutado** — el acuse de que el proceso corrió como estaba
     configurado.
   - **Expediente de evidencia** — exportable, con filtros y versión normativa.
3. **Vocabulario prohibido** en interfaz, documentos generados, contrato y material comercial:
   "certificación de cumplimiento", "empresa certificada", "entidad certificada", "avalado
   por la plataforma" y equivalentes. Ni siquiera como nombre de reporte (descarta la
   opción C).
4. **Disclaimer obligatorio** en toda salida: la plataforma informa y documenta; la decisión y
   la responsabilidad son del cliente.

## Consecuencias

- **Positivas:** el producto puede sostener lo que promete ante un supervisor; el glosario, la
  visión y los reportes hablan el mismo idioma; desaparece la exposición legal más obvia del
  negocio.
- **Negativas / costo asumido:** hay que renombrar el proyecto y rehacer el material que ya
  use el término. En la venta hay una fricción real: el comprador que pide "certificación"
  necesita una explicación de treinta segundos que antes no hacía falta.
- **Qué habría que revisar si el contexto cambia:** si alguna vez existiera un esquema de
  acreditación formal en Colombia para este tipo de servicio, y el negocio decidiera
  acreditarse, esta decisión se reabre — pero como una decisión de empresa, no de producto.

## Trazabilidad

- Cierra `PA-003` y `PA-022`. Confirma `SUP-007`.
- Afecta: `00-contexto/vision.md`, `00-contexto/glosario.md`, `HU-044` (exportación de
  reportes), `HU-016` (expediente reconstruible) y todo el material comercial.
