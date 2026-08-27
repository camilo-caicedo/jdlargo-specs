---
id: CTX-glosario
estado: propuesto
actualizado: 2026-08-27
---

# Glosario (lenguaje ubicuo)

Fuente de verdad terminológica del proyecto. Si un término no está aquí, no debería
aparecer en una historia de usuario.

Formato: **Término** — definición acordada. `(por validar)` mientras no lo confirme el cliente.

## Dominio

Definiciones tomadas del
[documento funcional del cliente](../01-descubrimiento/entregables-cliente/2026-08-21-flujo-plataforma-debida-diligencia-v2.md).
Cierra parcialmente `PA-003`.

### Los cinco conceptos que nunca se mezclan (§2)

Es la distinción central de todo el producto. Ver `ADR-0005`.

- **Recolección** — lo dijo la contraparte (lo que escribió o subió).
- **Extracción** — lo leyó la IA de un documento.
- **Verificación** — lo confirmó una fuente externa autorizada.
- **Evaluación** — el sistema aplicó una regla o fórmula a esos datos.
- **Decisión** — una persona autorizada, con nombre y cargo, decidió.

### Entidades del proceso

- **Contraparte** — la persona o empresa a la que se le hace debida diligencia. Es el término
  que usa el cliente; sustituye al genérico "entidad".
- **Persona relacionada** — quien "cuelga" de una contraparte: representante legal,
  beneficiario final, accionista, apoderado, conductor, propietario de un vehículo.
- **Beneficiario final** — quién realmente controla o se beneficia de una empresa, aunque no
  aparezca como accionista directo. **No es lo mismo que accionista**, y no se determina solo
  con el certificado de cámara de comercio.
- **Expediente** — el ciclo de vida completo de una vinculación, con toda su evidencia. Es la
  unidad central del sistema.
- **Solicitud de vinculación** — lo que abre un expediente.
- **PEP** — Persona Expuesta Políticamente: ocupa u ocupó un cargo público relevante, lo que
  exige mayor cuidado.
- **Caso** — toda alerta se convierte en un caso, que agrupa alerta, evidencia, análisis,
  decisión y cierre. **No se puede cerrar sin justificación registrada.**

### Del proceso de cumplimiento

- **Debida diligencia (DD)** — el proceso de conocimiento de la contraparte.
- **Debida diligencia intensificada (DDI)** — versión reforzada, que se activa por PEP,
  jurisdicción de alto riesgo, coincidencia confirmada, estructura societaria compleja,
  beneficiario final poco claro, información inconsistente o riesgo alto.
- **Matriz de requisitos** — la tabla configurable que define qué campo, documento y fuente
  aplica a cada combinación de estándar y tipo de contraparte. El corazón del producto.
- **Screening** — consulta a listas, PEP, sanciones, antecedentes y fuentes configuradas.
- **Matching** — comparación de nombres. Una **coincidencia técnica no es una coincidencia
  confirmada**: los estados son sin coincidencia, posible, descartada, confirmada, pendiente.
- **Riesgo inherente / residual** — antes y después de aplicar controles.
- **Monitoreo continuo** — vigilancia de cambios después de la vinculación. No se limita a
  volver a consultar listas.
- **Oficial de cumplimiento** — el responsable legal del proceso dentro del cliente.

### Términos que el documento redefine

- ~~**Validación / verificación / certificación**~~ — el nombre inicial del proyecto hablaba
  de "validación, verificación y certificación de entidades". El documento del cliente **no
  usa "certificación"**: el producto no certifica nada, automatiza y traza el proceso de
  debida diligencia. Ver §39 y `PA-022`.

### Del sustrato del sistema

Términos que no vienen del proceso de cumplimiento sino de cómo está construida la
plataforma. Necesarios desde `EP-000`.

- **Organización cliente** — la empresa que contrata la plataforma, vista como unidad de
  aislamiento de datos. Es el *tenant* de la §31: usuarios propios, expedientes
  independientes, configuración y bitácora propias. Toda fila del dominio pertenece a
  exactamente una.
- **Aislamiento entre organizaciones** — la garantía de que ningún dato de una organización
  cliente es legible ni modificable desde otra. No es una convención de código: es una
  política de la base de datos.
- **Afirmación** — la unidad de dato del expediente. Un valor **con su procedencia**: quién
  o qué lo produjo, cuándo, con qué evidencia y con qué origen. Un expediente no guarda
  valores, guarda afirmaciones (`ADR-0005`).
- **Origen (procedencia)** — de dónde viene una afirmación. Cuatro valores, tomados de los
  cinco conceptos de la §2: `declarado` (lo dijo la contraparte), `extraído` (lo leyó la
  IA), `verificado` (lo confirmó una fuente externa) y `evaluado` (lo produjo una regla).
  La **decisión** no es un origen: es un evento aparte, con persona identificada.
- **Versión de configuración** — el paquete publicable e inmutable de configuración de
  cumplimiento de una organización cliente (estándares, matriz de requisitos, metodología,
  reglas, formularios). No se edita: se publica una versión nueva y la anterior queda
  intacta (`ADR-0004`).
- **Configuración vigente** — la versión de configuración que se aplica a lo que empiece
  desde ahora. Es un puntero que se mueve, nunca una edición de la versión anterior.
- **Sujeto** — la persona u organización sobre la que se afirma algo: la contraparte del expediente o una persona relacionada con ella. Vive a nivel de organización cliente y se reutiliza entre expedientes del mismo cliente, nunca entre clientes distintos (§31, §46).
- **Bitácora** — el registro de auditoría transversal, de solo inserción: quién, qué,
  cuándo, desde dónde, valor anterior y nuevo, motivo, fuente, si fue automático o manual,
  qué modelo de IA intervino y qué versión de regla estaba vigente (§23). No es un módulo:
  es el sustrato.

### Del expediente y su recorrido

Términos que aparecen a partir de `EP-001`. Los cinco conceptos de la §2 siguen mandando: lo
que la contraparte escribe es `declarado`, nada más.

- **Tipo de contraparte** — la clasificación configurable que determina qué se le exige a una
  contraparte: cliente, proveedor, contratista, empleado, accionista, conductor,
  transportadora aliada, intermediario, tercero pagador. Vive en la configuración, no en el
  código (`ADR-0004`).
- **Enlace de acceso** — la dirección de un solo uso previsto con la que la contraparte entra a
  su expediente. Lleva asociado un **token de acceso** acotado a **un único expediente**, con
  expiración y revocable. La contraparte no tiene cuenta ni contraseña.
- **Aviso de privacidad** — el texto, configurable y versionado por cada organización cliente,
  que informa al titular qué datos se tratan, con qué finalidad, quién responde por ellos y
  cómo ejercer sus derechos.
- **Consentimiento** — la aceptación del titular cuando la base jurídica la exige, registrada
  con la versión exacta del aviso que aceptó, la fecha, la hora y el medio. Un aviso mostrado
  no es un consentimiento otorgado, y no todo tratamiento necesita el mismo tipo de
  autorización (§5).
- **Formulario dinámico** — el formulario que se arma en el momento a partir de la matriz de
  requisitos: qué campos, cuáles obligatorios y con qué validaciones sale de la configuración
  vigente, nunca de una pantalla escrita a mano (`ADR-0004`).
- **Tipo documental** — la clase de documento que la matriz de requisitos puede exigir (cédula,
  certificado de existencia y representación legal, RUT, estado financiero…), con sus reglas de
  emisor y vigencia.
- **Condición de la decisión** — la exigencia con la que se aprueba una vinculación cuando no se
  aprueba sin reservas. Forma parte de la decisión y es inmutable como ella.
- **Vigencia de la vinculación** — hasta cuándo vale una decisión antes de exigir actualización.

## Siglas

- **LA/FT** — Lavado de Activos / Financiación del Terrorismo.
- **FPADM** — Financiación de la Proliferación de Armas de Destrucción Masiva.
- **SARLAFT** — Sistema de Administración del Riesgo de LA/FT.
- **SAGRILAFT** — Sistema de Autocontrol y Gestión del Riesgo Integral de LA/FT/FPADM.
- **PTEE** — Programa de Transparencia y Ética Empresarial.
- **UIAF** — Unidad de Información y Análisis Financiero.
