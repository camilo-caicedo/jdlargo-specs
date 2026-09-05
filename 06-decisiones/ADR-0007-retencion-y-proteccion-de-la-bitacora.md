---
id: ADR-0007
titulo: Retención de la evidencia y nivel de protección de la bitácora
estado: propuesto
fecha: 2026-09-05
reemplaza-a: —
reemplazado-por: —
---

# ADR-0007 — Retención y protección de la bitácora

## Contexto

Dos preguntas que parecían separadas resultaron ser la misma decisión: **cuánto tiempo se
conserva la evidencia** (`PA-009`) y **qué tan protegido tiene que estar el registro que la
sostiene** (`PA-026`). Ninguna de las dos se puede añadir después sin rehacer trabajo.

Lo que aportó el cliente:

- El expediente de debida diligencia se conserva **10 años** (Art. 28 Ley 962/2005
  `(por validar)`). Su propuesta es que **cada cliente conserve su propio respaldo**, con la
  obligación pactada en el contrato y un correo de contacto al que la plataforma envía copias
  periódicas. Queda por evaluar si el negocio puede sostener 10 años de almacenamiento.
- Sobre la bitácora: *"Solamente por permisos de bases de datos, no requerimos algo tan
  complejo inicialmente."* La §23 admite "inmutable **o** técnicamente protegida".

## Opciones consideradas

### Opción A — Bitácora mutable ahora, endurecer después
- A favor: es lo más rápido de construir.
- En contra: descartada de plano. Endurecer una bitácora que ya aceptó `UPDATE` y `DELETE`
  obliga a rehacerla entera, y todo lo escrito antes queda sin garantía. Es exactamente el
  fallo que `HU-006` señala como irrecuperable.

### Opción B — Solo inserción por permisos de base de datos
- A favor: barato, suficiente para el MVP, es lo que el cliente pidió.
- En contra: por sí solo no prueba nada ante un tercero. Un administrador con acceso directo
  al motor puede alterar filas sin dejar rastro.

### Opción C — Encadenamiento criptográfico / almacenamiento WORM desde el día uno
- A favor: es el nivel de prueba que exige un régimen estricto.
- En contra: sobredimensiona el MVP en costo e infraestructura para un requisito que el
  cliente no pidió y que hoy nadie le exige.

## Decisión

**Opción B, construida de forma que la C sea una evolución y no una reescritura.**

### 1. Bitácora *append-only* desde la primera tabla

- **La aplicación no ejecuta `UPDATE` ni `DELETE` sobre la bitácora.** Nunca, en ningún caso.
- Se refuerza en el motor: permisos de base de datos que solo conceden `INSERT` y `SELECT` al
  rol de la aplicación, con la escritura en la misma transacción que el cambio que describe.
- **Segregación de permisos**: quien opera la aplicación no es quien administra el motor.
- **Hash por evento** desde el día uno. Es barato y es lo que convierte el endurecimiento
  posterior en una migración y no en una reescritura.
- Respaldos protegidos, con su propia política de acceso.

### 2. WORM y hash encadenado quedan como evolución, no como MVP

Se activan por plan o por régimen cuando un cliente lo exija. Como el hash por evento ya
existe, encadenarlo es añadir una columna y un proceso de verificación — no rehacer la
bitácora.

### 3. La retención es configurable, y su política es contractual antes que técnica

- **No se promete una retención universal.** El sistema conserva evidencia suficiente para
  reconstruir el proceso; **cuánto tiempo** lo fija una política por organización cliente,
  validada jurídicamente en su contrato.
- Referencia de partida: **10 años** para el expediente de debida diligencia `(por validar)`.
- **El cliente conserva su propio respaldo.** La plataforma envía copias periódicas al
  contacto que él designe, y esa obligación queda escrita en el contrato — no en un correo.
- La retención tiene que armonizar cuatro cosas que no siempre apuntan igual: obligación de
  cumplimiento, protección de datos personales (Ley 1581 de 2012 `(por validar)`), contrato y
  política interna del cliente. Donde choquen, decide el contrato, no el software.

### 4. Qué registra cada entrada

Quién, qué, cuándo, desde dónde, origen, valor anterior y nuevo, motivo, fuente, si fue
proceso automático o manual, y la versión de regla, norma o modelo de IA vigente.
**La bitácora no se depura** mientras la política de retención de esa organización no lo
autorice explícitamente.

## Consecuencias

- **Positivas:** el MVP no carga con infraestructura WORM; la ruta al nivel alto de protección
  queda abierta y es incremental; la exposición por almacenar 10 años se reparte con el
  cliente en vez de recaer entera en el proveedor.
- **Negativas / costo asumido:** la protección real depende de disciplina operativa
  (permisos, segregación, respaldos), que es más fácil de romper que un control criptográfico.
  Hay que auditarla de verdad, no darla por hecha. Y el envío periódico de copias al cliente
  es un proceso que hay que construir y monitorear: si falla en silencio, el respaldo que
  creíamos delegado no existe.
- **Qué habría que revisar si el contexto cambia:** un cliente en régimen que exija
  inmutabilidad demostrable, o un incidente de integridad, activan el hash encadenado y WORM.
  Está previsto, no improvisado.

## Trazabilidad

- Cierra `PA-009` y `PA-026`. Deriva `PA-044` (decidir si el negocio asume los 10 años de
  almacenamiento o los delega por contrato).
- Afecta: `HU-006`, `HU-013`, `HU-016`, `HU-044`, `03-requisitos/no-funcionales.md`.
