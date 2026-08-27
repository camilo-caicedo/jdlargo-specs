---
id: HU-033
titulo: Debida diligencia intensificada
estado: borrador
epica: EP-004
prioridad: Must
actualizado: 2026-08-27
---

# HU-033 — Debida diligencia intensificada

## Historia

**Como** Oficial de Cumplimiento
**quiero** activar una diligencia reforzada y pedir desde la propia plataforma lo que necesito
para decidir
**para** no tener que salirme a correos y llamadas justo en los expedientes que más riesgo tienen
y más evidencia exigen.

## Contexto

Es la Fase 16. Se activa cuando la persona es PEP, la jurisdicción es de alto riesgo, hay una
coincidencia confirmada, la estructura societaria es compleja, el beneficiario final no está
claro, la información es inconsistente, aparecen señales de alerta, las operaciones son atípicas,
el riesgo calculado es alto, **o por cualquier otra regla que el cliente configure**.

Lo que el Oficial de Cumplimiento puede solicitar, todo gestionado sin salir de la plataforma:
información adicional, soportes del origen de los recursos, soportes financieros, documentos
societarios adicionales, información sobre el beneficiario final, referencias comerciales o
personales, una **visita presencial** con lista de comprobación digital —fotos, notas, firma de
quien visita—, una entrevista, la explicación de la operación, la aprobación de la alta gerencia,
o controles adicionales.

Y el límite, que el documento marca en negrita:

> **La plataforma gestiona la debida diligencia intensificada — no determina por sí sola qué
> evidencia es jurídicamente suficiente.**

## Criterios de aceptación

```gherkin
Escenario: La diligencia intensificada se activa por una regla configurada
  Dado una metodología que define que una coincidencia confirmada la exige
  Cuando se confirma una coincidencia en un expediente
  Entonces el expediente queda marcado como sujeto a debida diligencia intensificada
  Y queda registrado qué regla la activó y con qué versión de la metodología
```

```gherkin
Escenario: Activación manual con fundamento
  Dado un expediente sin causal automática
  Cuando el Oficial de Cumplimiento activa la diligencia intensificada indicando el motivo
  Entonces queda registrada la activación con su nombre, su cargo, el momento y el motivo
```

```gherkin
Escenario: Solicitar evidencia adicional a la contraparte
  Dado un expediente en debida diligencia intensificada
  Cuando el Oficial de Cumplimiento solicita soportes del origen de los recursos
  Entonces la solicitud queda registrada con lo que se pide y por qué
  Y la contraparte puede aportarlos por su enlace de acceso
  Y lo aportado se registra con su procedencia, como cualquier otro documento
```

```gherkin
Escenario: Registrar una visita presencial
  Dado una diligencia intensificada que exige visita
  Cuando quien la realiza completa la lista de comprobación desde la plataforma
  Entonces quedan registrados el momento, las respuestas, las fotografías, las notas y la firma de quien visitó
  Y la evidencia queda asociada al expediente
```

```gherkin
Escenario: Registrar una aprobación de alta gerencia
  Dado una diligencia intensificada que exige aprobación de alta gerencia
  Cuando quien tiene ese permiso la otorga
  Entonces queda registrada con su nombre, su cargo, el momento y su fundamento
  Y el expediente no puede llegar a decisión sin ella si la configuración la exige
```

```gherkin
Escenario: La plataforma no juzga la suficiencia de la evidencia
  Dado un expediente en diligencia intensificada con parte de lo solicitado aportado
  Cuando se consulta su estado
  Entonces el sistema muestra qué se solicitó, qué se aportó y qué falta
  Y no emite ningún juicio sobre si lo aportado es jurídicamente suficiente
```

```gherkin
Escenario: Cerrar la diligencia intensificada
  Dado una diligencia intensificada con sus solicitudes atendidas
  Cuando el Oficial de Cumplimiento la da por concluida indicando su conclusión
  Entonces queda registrada con quién la concluyó, cuándo y con qué fundamento
  Y el expediente puede avanzar a decisión
  Y toda la evidencia recogida queda como parte del expediente
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la diligencia intensificada
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta expedientes en diligencia intensificada con su contexto de usuario propagado
  Entonces obtiene únicamente los de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Las causales de activación son **configuración** (`ADR-0004`), no una lista fija en el código.
  Las de la Fase 16 son el punto de partida.
- La activación, sea automática o manual, queda registrada con su causa y la versión de
  metodología aplicable.
- Todo lo que se solicita y todo lo que se aporta queda registrado con su procedencia
  (`ADR-0005`). Una referencia comercial es una afirmación declarada; un documento societario
  adicional, un documento más.
- La visita presencial se completa **desde la plataforma**: lista de comprobación, fotografías,
  notas y firma de quien visita.
- La aprobación de alta gerencia es un permiso más de la matriz (`HU-003`), con su registro
  propio.
- **La plataforma no determina si la evidencia es jurídicamente suficiente.** Muestra qué se pidió
  y qué llegó; el juicio es del cliente (§40).
- Concluir la diligencia intensificada exige fundamento registrado y no equivale a decidir sobre
  la vinculación (`HU-015`).

## Fuera de alcance

- La decisión sobre la vinculación → `HU-015`.
- El cálculo del riesgo que puede activarla → `HU-032`.
- La diligencia estándar, que es el camino directo a decisión cuando no hay causales (Fase 15):
  no necesita historia propia, es la ausencia de esta.
- Las visitas como módulo de agenda o de rutas.
- La valoración de la calidad de las referencias aportadas.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `ddi.organization_id` | Sí | Organización cliente existente | No |
| `ddi.expediente_id` | Sí | Expediente de la misma organización cliente | No |
| `ddi.causal` | Sí | Regla configurada que la activó, o `manual` | No |
| `ddi.activada_por` | Condicional | Obligatorio si la activación fue manual | No |
| `ddi.motivo` | Condicional | Obligatorio si la activación fue manual | No |
| `ddi.metodologia_version_id` | Sí | Versión vigente al activarla | No |
| `solicitud_ddi.tipo` | Sí | `informacion` \| `origen_recursos` \| `soportes_financieros` \| `documentos_societarios` \| `beneficiario_final` \| `referencias` \| `visita` \| `entrevista` \| `explicacion_operacion` \| `aprobacion_alta_gerencia` \| `controles` | No |
| `solicitud_ddi.estado` | Sí | `pendiente` \| `atendida` \| `no_atendida` | No |
| `visita.lista_comprobacion` | Condicional | Obligatoria si hay visita; respuestas registradas | Sí |
| `visita.firma_visitante` | Condicional | Obligatoria si hay visita | Sí |
| `ddi.conclusion` y `concluida_por` | Condicional | Obligatorias al concluir | No |

## Trazabilidad

- Épica: `EP-004`
- Capacidad: `CAP-04`
- Documento del cliente: Fase 16, Fase 15, §40, §44 pregunta K
- Decisiones: `ADR-0004` (causales configurables), `ADR-0005` (todo lo aportado lleva procedencia)

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-018` (qué causales usa el primer cliente), `PA-029` (si un
  expediente con diligencia intensificada incompleta puede llegar a decisión). **Queda en
  `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-032` (riesgo), `HU-026` (coincidencias confirmadas), `HU-030` (beneficiario
  final poco claro), `HU-013` (documentos), `HU-010` (la contraparte aporta por su enlace).
- **Habilita a:** `HU-015` sobre expedientes de riesgo alto, y el monitoreo reforzado de la Fase 6.
- **Riesgo:** es la parte del producto que más se parece a un gestor de tareas y donde más tienta
  automatizar el criterio. La plataforma organiza la diligencia; decidir si lo aportado alcanza es
  del Oficial de Cumplimiento, y el producto no debe insinuar lo contrario.
