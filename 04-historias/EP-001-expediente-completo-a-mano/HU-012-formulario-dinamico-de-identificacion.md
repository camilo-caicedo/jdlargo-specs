---
id: HU-012
titulo: Formulario dinámico de identificación
estado: borrador
epica: EP-001
prioridad: Must
actualizado: 2026-08-27
---

# HU-012 — Formulario dinámico de identificación

## Historia

**Como** Contraparte
**quiero** que el formulario me pida solo lo que aplica a mi caso y me lo guarde a medida que
avanzo
**para** entregar la información una sola vez y sin tener que adivinar qué es relevante para mí.

## Contexto

Es la Fase 6 del documento del cliente, y `ADR-0004` extrae de ella una consecuencia técnica
concreta: **los esquemas de validación del expediente no pueden ser estáticos**. Los campos, su
obligatoriedad, sus validaciones y su condicionalidad salen de la matriz de requisitos
(`HU-007`), así que hace falta armar el esquema en tiempo de ejecución desde la configuración
almacenada. No hay una pantalla por tipo de contraparte: hay una pantalla que se arma sola.

Y la regla que gobierna lo que se guarda viene de `ADR-0005`: **lo que la contraparte escribe
es una afirmación de origen `declarado`**, con su autor y su momento. No es un hecho verificado
y el sistema no debe presentarlo como tal. Verificarlo es la Fase 3.

El documento del cliente añade una advertencia que conviene tener presente: *"el formulario es
un insumo, no el proceso completo"* (§34).

## Criterios de aceptación

```gherkin
Escenario: El formulario se arma desde la matriz de la versión citada
  Dado un expediente de tipo "proveedor" que cita la versión 3 de configuración
  Cuando la contraparte abre el formulario
  Entonces ve exactamente los campos que la matriz de la versión 3 exige a ese tipo
  Y cada campo muestra su obligatoriedad y sus validaciones según esa matriz
  Y no ve ningún campo que la matriz no exija
```

```gherkin
Escenario: Dos tipos de contraparte ven formularios distintos
  Dado dos expedientes de la misma organización cliente, uno de tipo "proveedor" y otro de tipo "conductor"
  Cuando cada contraparte abre su formulario
  Entonces cada una ve el conjunto de campos que su tipo exige
  Y la diferencia proviene de la matriz, no de una pantalla escrita para cada tipo
```

```gherkin
Escenario: Un campo condicional aparece solo cuando corresponde
  Dado una matriz que exige el campo "detalle del cargo público" solo si la contraparte declara ser PEP
  Cuando la contraparte responde que no es PEP
  Entonces el campo condicional no se le solicita
  Y cuando cambia su respuesta a que sí es PEP
  Entonces el campo condicional aparece y pasa a ser obligatorio
```

```gherkin
Escenario: Lo que se guarda es una afirmación declarada
  Dado una contraparte que diligencia el campo razon_social con el valor "Ficticia S.A.S."
  Cuando guarda el formulario
  Entonces se registra una afirmación con origen declarado, con quién la produjo y cuándo
  Y no se escribe ese valor como un dato plano del expediente
  Y el sistema no presenta esa información como verificada en ninguna pantalla
```

```gherkin
Escenario: Corregir antes de enviar añade, no reemplaza
  Dado una contraparte que ya guardó el campo razon_social con un valor
  Cuando lo corrige y vuelve a guardar
  Entonces queda registrada una afirmación nueva con el valor corregido
  Y la afirmación anterior sigue existiendo con su momento y su autor
  Y la corrección queda registrada en la bitácora
```

```gherkin
Escenario: El avance parcial no se pierde
  Dado una contraparte que diligenció parte del formulario y cerró el navegador
  Cuando vuelve a entrar con su enlace de acceso vigente
  Entonces recupera lo que ya había guardado
  Y puede continuar desde donde quedó
```

```gherkin
Escenario: No se puede enviar el formulario incompleto
  Dado un formulario con campos obligatorios sin diligenciar
  Cuando la contraparte intenta darlo por terminado
  Entonces la operación es rechazada indicando qué campos faltan
  Y el expediente no cambia de estado
```

```gherkin
Escenario: Las validaciones vienen de la configuración, no del programa
  Dado una matriz que define para un campo un formato determinado
  Cuando la contraparte escribe un valor que no cumple ese formato
  Entonces el valor es rechazado con el mensaje que corresponde a esa validación
  Y la misma validación se aplica en el servidor, no solo en el navegador
```

## Reglas de negocio

- El formulario se **construye** a partir de la matriz de requisitos de la versión de
  configuración que cita el expediente. Ningún campo está escrito en la pantalla.
- Toda respuesta se guarda como **afirmación de origen `declarado`** (`HU-005`), con autor y
  momento. Nunca como valor plano.
- Corregir es añadir una afirmación nueva; la anterior no se borra.
- Las validaciones se aplican **en el servidor**. Lo que valide el navegador es comodidad, no
  control.
- La condicionalidad se expresa con el conjunto cerrado de condiciones de `ADR-0004`. No hay
  lógica condicional escrita en el programa para un cliente concreto.
- El formulario se guarda parcialmente y se puede retomar mientras el enlace de acceso siga
  vigente (`HU-010`).
- Dar el formulario por terminado es una **transición** del expediente (`HU-009`), no una marca
  en una columna.
- Lo declarado no es un hecho verificado, y la interfaz no debe sugerir lo contrario: es la
  regla de la §2 aplicada a la pantalla.

## Fuera de alcance

- La extracción con IA que autollena el formulario y la conciliación de lo declarado con lo
  extraído → Fase 2.
- La verificación externa de lo declarado → Fase 3.
- La firma electrónica del formulario → Fase 2.
- La plantilla propia del sector transporte: es un caso de uso de la matriz (`HU-007`), no una
  pantalla aparte.
- Los datos de personas relacionadas y del beneficiario final más allá de lo que la matriz pida
  como campo declarado. Su tratamiento propio llega en la Fase 4.
- Traducción del formulario a otros idiomas.

## Datos y validaciones

Los campos concretos **los define la matriz de requisitos**, no esta historia. Como referencia,
el documento del cliente enumera en su Fase 6 lo típico de cada naturaleza:

| Naturaleza | Campos de referencia (configurables, no fijos) | Sensible |
|---|---|---|
| Persona natural | Nombres y apellidos, tipo y número de documento, fecha y lugar de nacimiento, nacionalidad, actividad económica, dirección, teléfono, correo, información financiera, propósito de la relación, origen de los recursos, condición de PEP | Sí |
| Persona jurídica | Razón social, identificación tributaria, país, actividad económica, domicilio, dirección, teléfono, correo, representante legal, propósito de la relación, origen de los recursos, estructura de propiedad, beneficiario final declarado, información bancaria, jurisdicciones donde opera | Sí |

Reglas que sí son de esta historia:

| Regla | Validación |
|---|---|
| Todo campo diligenciado | Se guarda como afirmación con origen `declarado` |
| Obligatoriedad | Sale de la matriz; se verifica en el servidor |
| Condicionalidad | Solo con el conjunto cerrado de condiciones de `ADR-0004` |
| Tipos de dato y formatos | Salen de la matriz; el programa no conoce formatos por campo |
| Datos personales | Todo campo de identificación se trata como sensible a efectos de registro y acceso |

## Trazabilidad

- Épica: `EP-001`
- Capacidad: `CAP-01`
- Documento del cliente: Fase 6, §2, §34, §44 pregunta B
- Decisiones: `ADR-0004` (formularios dinámicos, esquemas no estáticos), `ADR-0005`
  (afirmaciones declaradas)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-018` — cuántos tipos de contraparte hay que soportar, que determina
  cuánta variedad tiene que absorber el constructor del formulario. Queda en `borrador` también
  por arrastre de `HU-007`.
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-007` (la matriz), `HU-008` (el expediente y su versión citada), `HU-010`
  (la contraparte entró), `HU-011` (el aviso se resolvió primero), `HU-005` (afirmaciones).
- **Habilita a:** `HU-013`, `HU-014`, y en la Fase 2 la conciliación entre lo declarado y lo
  extraído.
- **Riesgo:** es la pieza de interfaz más compleja del producto y la que más tienta a escribirse
  a mano "solo por esta vez, para el primer cliente". Un formulario escrito a mano es una
  bifurcación por cliente y rompe `ADR-0004` en el punto donde más cuesta volver atrás.
