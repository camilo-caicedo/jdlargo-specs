---
id: DESC-hallazgos-gap-analysis
titulo: Hallazgos del análisis de brechas post-respuestas de Juan David
estado: vivo
actualizado: 2026-09-05
---

# Hallazgos del análisis de brechas (2026-09-05)

> Nota de cierre de la segunda tanda de trabajo tras las respuestas de Juan David a `PA-001`–
> `PA-039`. Documenta qué se encontró al aplicar un barrido sistemático de seis dimensiones
> (funcional, no funcional, escenarios de usuario, contexto de negocio, contexto técnico,
> atributos de calidad) más una revisión explícita de contradicciones — el método que usa
> `awslabs/aidlc-workflows` en su etapa de *Requirements Analysis*, aplicado aquí a mano y
> directamente sobre `jdlargo-specs`, sin instalar el framework (ver discusión en el hilo de
> trabajo: AI-DLC es code-forward para *brownfield* y trata los documentos existentes como no
> autoritativos, así que no encajaba con un repo de specs ya maduro; su método de análisis sí).

## 1. El hueco más grande: requisitos funcionales sin escribir

`03-requisitos/funcionales.md` llevaba meses en `TBD` mientras las 50 historias de usuario ya
estaban escritas — la cadena `capacidad → requisito → historia` no existía en ningún documento.

**Cerrado:** se escribieron `RF-001` a `RF-050`, uno por historia (numeración 1:1 deliberada),
derivados de releer la sección "Historia" y "Reglas de negocio" de las 50 `HU-xxx`. Cada uno
cita su capacidad, su origen (sección del documento de Juan David, ADR o PA que lo sustenta),
prioridad heredada del backlog y estado `Borrador` (ninguno pasa a `en-revision` hasta que su
historia complete la *Definition of Ready*).

Dos huecos quedan explícitos dentro del archivo, no resueltos por silencio:
- `CAP-08` (salida de datos hacia otros sistemas, `PA-010`) no tiene épica ni historias todavía,
  así que no tiene `RF-xxx`.
- `RF-012` y `RF-017` (el orden formulario/documentos) se redactaron sobre un supuesto de que
  ambos caminos son posibles, porque `PA-042` (ver abajo) sigue abierta.

## 2. Contradicciones encontradas y corregidas

Al releer las 50 historias completas para derivar los `RF-xxx`, apareció una familia de
referencias que el barrido de propagación anterior no había tocado: quedaron ancladas al
supuesto **`SUP-005`** ("todo es una organización, el usuario individual es una organización de
un miembro"), que la actualización de `supuestos.md` ya había marcado como **derribado y
reemplazado** por el modelo confirmado `Usuario (identidad global) › Membresía › Organización ›
Roles` (`PA-013`). Se corrigieron los cinco puntos donde el texto seguía afirmando el modelo
viejo como si estuviera vigente:

- `ADR-0001-stack-tecnico.md` — el punto 7 de la decisión de identidad, que colisionaba de frente
  con el punto 9 de la misma decisión (que sí reflejaba el modelo nuevo).
- `04-historias/EP-000-cimientos/HU-001-organizaciones-cuentas-y-pertenencia.md` — la propia
  historia que define identidad tenía la nota de actualización correcta al principio, pero la
  sección "Contexto" y "Dependencias" seguían citando `SUP-005` como decisión vigente.
- `04-historias/EP-000-cimientos/EP-000-cimientos.md` — mención en "Supuestos" de la épica.
- `04-historias/EP-005-el-cliente-se-autogestiona/HU-038-administrar-usuarios-roles-y-permisos.md`
  — mención en "Supuestos".
- `04-historias/EP-007-capa-comercial/HU-046-planes-licencias-y-cupo-de-consultas.md` — regla de
  negocio que afirmaba el modelo viejo sin ninguna nota de que había cambiado.

Una segunda contradicción, de la misma familia (un documento de resumen sin actualizar tras una
respuesta de Juan David que sí se propagó a la historia detallada): `04-historias/EP-007-capa-
comercial/EP-007-capa-comercial.md` seguía describiendo el cobro como "patrón dual: tarjeta
tokenizada para clientes pequeños, factura para empresas" en tres lugares (objetivo, alcance y
riesgo), mientras que `HU-049-cobro-por-pasarela.md` ya reflejaba correctamente que `PA-016`
eliminó por completo la tarjeta tokenizada a favor de una única vía (factura + enlace de pago
para todos). Corregido en los tres lugares.

**Dónde quedó cada `SUP-005` que sigue en el texto:** los cuatro que sobreviven (`supuestos.md`,
`HU-001`, `HU-046`, `ADR-0001`) ahora dicen explícitamente "derribado y reemplazado" — se dejaron
a propósito como registro histórico de qué se descartó y por qué, no como afirmación activa.

## 3. Barrido adicional (sin hallazgos nuevos)

Se revisaron sin encontrar más contradicciones: menciones a "certificación" (todas son el
recordatorio del término prohibido, `ADR-0006`), terminología de "entidad/contraparte" en el
glosario (ya no queda "por confirmar"), y "inicio de sesión único" (una sola mención, coherente
con `ADR-0001`). Los `TBD` que quedan en el repo (`normativa/*.md`, `HU-046`, `no-funcionales.md`,
algunas historias puntuales) son huecos ya registrados como preguntas o supuestos explícitos, no
huecos silenciosos — se dejaron como están.

## 4. Investigación de `PA-040` y `PA-045`

Ninguna de las dos preguntas se cierra del todo — ambas requieren una cotización comercial o una
confirmación jurídica formal que no corresponde inventar aquí — pero la investigación adelantó
material concreto para quien las cierre. El detalle completo, con fuentes, queda en las filas de
`preguntas-abiertas.md`; resumen:

- **`PA-040`** (costo real de fuentes colombianas): la pregunta original asumía dos vías —
  conexión directa a cada fuente, o "un intermediario". La investigación encontró una tercera
  vía que ni `SUP-010` ni Juan David habían nombrado: agregadores colombianos de dominio
  LA/FT/KYC ya construidos para este caso exacto (Tusdatos, Verdata, RiskTech,
  Compliance.com.co), que combinan RUES, antecedentes judiciales/disciplinarios/fiscales y
  listas restrictivas en una sola API. Ninguno publica precio: sigue sin cotización real, pero el
  siguiente paso deja de ser "cotizar con cada entidad pública" y pasa a ser "pedir cotización a
  2-3 de estos agregadores".
- **`PA-045`** (proveedor de IA): la Superintendencia de Industria y Comercio (Circular 002 de
  2018) lista a Estados Unidos entre los países con nivel adecuado de protección de datos
  (junto con Alemania, Costa Rica, Francia, Italia, México y Perú) — si asesoría jurídica lo
  confirma vigente (pendiente, por la misma razón que `SUP-008`: normas citadas sin verificar
  aquí), procesar datos en un proveedor de IA con sede en EE. UU. no exigiría cláusulas
  contractuales adicionales. Sobre el proveedor: Anthropic (Claude) no entrena con datos de la
  API por defecto, ofrece retención cero de datos y control de región de inferencia; Azure OpenAI
  ofrece residencia de datos configurable por despliegue. Ambos quedan como candidatos viables
  sujetos a firmar el DPA correspondiente.

## 5. Qué no se tocó

- No se inventó ningún requisito, precio, plazo ni cifra donde Juan David o la ley no la han
  dado — cada `RF-xxx` cita su origen y cada hallazgo de `PA-040`/`PA-045` se presenta como
  investigación de apoyo, no como respuesta cerrada.
- No se cerraron `PA-040` ni `PA-045`: siguen `Abierta`, con responsable asignado, porque cerrarlas
  de verdad requiere una cotización o una firma jurídica que no le corresponde a este documento.
- No se tocó `01-descubrimiento/entregables-cliente/`: son los documentos crudos que entregó
  Juan David, se conservan como fuente primaria sin editar.
