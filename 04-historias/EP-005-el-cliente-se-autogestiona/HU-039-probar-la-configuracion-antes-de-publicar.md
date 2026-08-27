---
id: HU-039
titulo: Probar la configuración antes de publicar
estado: borrador
epica: EP-005
prioridad: Should
actualizado: 2026-08-27
---

# HU-039 — Probar la configuración antes de publicar

## Historia

**Como** Oficial de Cumplimiento
**quiero** ensayar mi configuración contra un caso de prueba y ver exactamente qué pediría y qué
riesgo calcularía
**para** no descubrir un error de configuración cuando ya lo estén sufriendo las contrapartes
reales.

## Contexto

Es la historia que evita que `EP-005` sea peligrosa. Poner la configuración en manos del cliente
sin darle forma de comprobarla es trasladarle el riesgo sin darle el instrumento.

`ADR-0004` ya dejó la puerta abierta: el evaluador es una **función pura** —recibe configuración y
datos, devuelve resultado más explicación, sin leer ni escribir en la base de datos— y eso es
justamente lo que permite ejecutarlo contra un expediente ficticio sin efectos secundarios. La
decisión técnica de entonces se cobra aquí.

Y con ella se cumple la advertencia del propio `ADR-0004`: el motor debe ser **depurable por el
cliente que escribe las reglas**, no solo por un desarrollador.

## Criterios de aceptación

```gherkin
Escenario: Ensayar la matriz con un caso ficticio
  Dado un borrador de configuración con su matriz de requisitos
  Cuando el Oficial de Cumplimiento define un caso de prueba con un tipo de contraparte y unos datos ficticios
  Entonces el sistema muestra qué campos y qué documentos se le exigirían
  Y muestra de qué regla de la matriz sale cada exigencia
```

```gherkin
Escenario: Ensayar la metodología de riesgo
  Dado un borrador con una metodología de riesgo
  Cuando se ejecuta el caso de prueba
  Entonces se muestra el nivel de riesgo que resultaría
  Y las reglas que se dispararon, con los datos de entrada de cada una
```

```gherkin
Escenario: El ensayo no toca datos reales
  Dado un ensayo ejecutado sobre un caso de prueba
  Cuando termina
  Entonces no se ha creado ningún expediente
  Y no se ha ejecutado ninguna consulta a fuentes externas
  Y no se ha generado ningún costo
```

```gherkin
Escenario: Comparar el resultado con la configuración vigente
  Dado un borrador con cambios respecto a la versión vigente
  Cuando se ejecuta el mismo caso de prueba contra ambas
  Entonces se muestra en qué difieren los requisitos exigidos y el nivel de riesgo
  Y se señala qué cambio de configuración provoca cada diferencia
```

```gherkin
Escenario: Detectar una regla que nunca se dispara
  Dado un borrador con una regla cuya condición no puede cumplirse con los datos que el expediente recoge
  Cuando se valida el borrador
  Entonces el sistema advierte que esa regla nunca se aplicará
  Y señala qué dato le falta
```

```gherkin
Escenario: Guardar casos de prueba para reutilizarlos
  Dado un caso de prueba definido
  Cuando el Oficial de Cumplimiento lo guarda
  Entonces queda disponible para ensayar futuras versiones de configuración
  Y se puede ejecutar el conjunto completo de casos guardados de una vez
```

```gherkin
Escenario: Datos ficticios obligatorios
  Dado un intento de crear un caso de prueba con datos de una contraparte real
  Cuando se guarda
  Entonces el sistema advierte que los casos de prueba deben usar datos ficticios
  Y el caso queda marcado como de prueba, sin mezclarse con expedientes reales
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre los ensayos
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando ejecuta o consulta casos de prueba con su contexto de usuario propagado
  Entonces solo accede a los de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- El ensayo ejecuta el **mismo evaluador** que la operación real. Un motor de prueba distinto no
  probaría nada.
- El ensayo **no produce efectos**: ni expediente, ni consultas externas, ni costo, ni entradas en
  la operación diaria. Es posible porque el evaluador es una función pura (`ADR-0004`).
- El resultado del ensayo incluye siempre **la explicación**: qué reglas se dispararon y con qué
  entradas.
- Los casos de prueba se guardan y se pueden ejecutar en conjunto contra cada versión nueva. Es la
  forma en que el cliente comprueba que un cambio no rompió lo anterior.
- Los casos de prueba usan **datos ficticios** y viven separados de los expedientes reales.
- La validación estática señala reglas inalcanzables y factores sin dato que los alimente.

## Fuera de alcance

- La ejecución real del expediente → `EP-001` a `EP-004`.
- Pruebas de carga o de rendimiento de la configuración.
- La generación automática de casos de prueba a partir de expedientes reales: implicaría copiar
  datos personales a un entorno de ensayo, y eso exige una decisión que el cliente no ha tomado.
- La comparación entre dos versiones antiguas cualesquiera.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `caso_prueba.organization_id` | Sí | Organización cliente existente | No |
| `caso_prueba.nombre` | Sí | Texto no vacío | No |
| `caso_prueba.tipo_contraparte` | Sí | Tipo de la versión que se ensaya | No |
| `caso_prueba.datos` | Sí | **Datos ficticios**; nunca de una contraparte real | No |
| `ensayo.version_configuracion_id` | Sí | Borrador o versión publicada que se ensaya | No |
| `ensayo.resultado` | Sí | Requisitos exigidos y nivel de riesgo | No |
| `ensayo.explicacion` | Sí | Reglas disparadas con sus entradas | No |
| `ensayo.efectos` | Sí | Ninguno: sin expediente, sin consultas, sin costo | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: §41, §42, §45
- Decisiones: `ADR-0004` (evaluador puro, comprobable con tablas de casos, depurable por el
  cliente)
- Reglas del repositorio: datos ficticios en los ejemplos

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-017` — si el cliente no configura por su cuenta, esta historia
  pierde buena parte de su razón de ser. **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-032` (el evaluador), `HU-034` (borradores), `HU-035`, `HU-036`.
- **Habilita a:** que `EP-005` se pueda entregar sin trasladar al cliente un riesgo que no puede
  controlar.
- **Riesgo:** si el ensayo se implementa con un evaluador simplificado "solo para probar", deja de
  demostrar nada y da una falsa confianza, que es peor que no tener ensayo.
