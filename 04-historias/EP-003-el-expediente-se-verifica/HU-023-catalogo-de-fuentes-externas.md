---
id: HU-023
titulo: Catálogo de fuentes externas
estado: borrador
epica: EP-003
prioridad: Must
actualizado: 2026-08-27
---

# HU-023 — Catálogo de fuentes externas

## Historia

**Como** Administrador de una organización cliente
**quiero** que las fuentes que se consultan estén registradas como configuración, con su
cobertura, sus condiciones de uso y su costo
**para** poder añadir o retirar una fuente sin que eso sea un cambio en el programa, y saber qué
me cuesta cada consulta antes de hacerla.

## Contexto

La Fase 11 lo dice sin ambigüedad: **debe existir un catálogo de fuentes, no fuentes fijas en el
código**. Cada fuente registra nombre, dirección de acceso, proveedor, tipo, país, cobertura,
frecuencia de actualización, condiciones de uso y costo.

Los tipos a contemplar son variados: registros empresariales, identidad, autoridades de
tránsito, otras autoridades, listas restrictivas, PEP, sanciones, antecedentes, medios de
comunicación y fuentes comerciales.

Hay una consecuencia económica que conviene tener escrita desde esta historia: la §31 prohíbe
reutilizar datos de contrapartes entre clientes del SaaS, de modo que **la deduplicación de
consultas solo puede darse dentro de una misma organización cliente**. Si dos clientes verifican
al mismo proveedor, se paga dos veces.

## Criterios de aceptación

```gherkin
Escenario: Registrar una fuente en el catálogo
  Dado una versión de configuración en borrador
  Cuando se registra una fuente con su nombre, proveedor, tipo, país, cobertura, condiciones de uso y costo por consulta
  Y se publica la versión
  Entonces la fuente queda disponible para verificación y screening
  Y ninguna referencia a esa fuente aparece escrita en el código del programa
```

```gherkin
Escenario: La matriz decide qué fuente aplica a qué requisito
  Dado una matriz de requisitos que asocia un campo con una fuente del catálogo
  Cuando se consulta qué debe verificarse en un expediente
  Entonces se obtiene qué campo se verifica contra qué fuente
  Y la asociación proviene de la configuración, no del programa
```

```gherkin
Escenario: Añadir una fuente no exige desplegar
  Dado un catálogo de fuentes publicado
  Cuando el Administrador publica una versión nueva que añade una fuente
  Entonces las consultas posteriores pueden usarla
  Y no fue necesario desplegar una versión nueva del programa
```

```gherkin
Escenario: Una fuente indisponible se registra como tal
  Dado una fuente configurada cuyo servicio no responde
  Cuando se intenta consultarla
  Entonces queda registrado el intento con la indisponibilidad, la fecha y la hora
  Y el dato que se pretendía verificar queda como no verificado, nunca como verificado
  Y no se inventa un resultado
```

```gherkin
Escenario: El costo de cada consulta queda registrado
  Dado una fuente con costo por consulta configurado
  Cuando se ejecuta una consulta contra ella
  Entonces queda registrado el costo de esa consulta asociado a la organización cliente
  Y ese registro sirve a la vez de evidencia de auditoría y de medición de consumo
```

```gherkin
Escenario: No hay reutilización entre organizaciones clientes
  Dado que "Alfa Ficticia S.A.S." ya consultó una fuente sobre la contraparte "Ficticia S.A.S."
  Cuando "Beta Ficticia S.A.S." necesita verificar a esa misma contraparte
  Entonces se ejecuta una consulta nueva e independiente
  Y no se reutiliza la respuesta obtenida por la otra organización cliente
  Y se registra el costo de ambas consultas por separado
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre el catálogo
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta el catálogo de fuentes con su contexto de usuario propagado
  Entonces obtiene únicamente las fuentes configuradas por "Alfa Ficticia S.A.S."
  Y un intento de modificar el catálogo de "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

## Reglas de negocio

- El catálogo de fuentes es **contenido de una versión de configuración** (`ADR-0004`). Se
  publica, no se edita.
- Cada fuente registra: nombre, proveedor, dirección de acceso, tipo, país, cobertura, frecuencia
  de actualización, condiciones de uso y costo por consulta (Fase 11).
- Las credenciales de acceso a cada fuente son **por organización cliente** (§31) y nunca llegan
  al navegador: toda consulta ocurre en el servidor.
- Una fuente indisponible produce un registro de indisponibilidad, no un resultado. **Un dato sin
  respuesta es no verificado**, que es distinto de no verificable.
- El costo por consulta se registra en el mismo evento que la evidencia (`ADR-0001` §18): una
  sola tabla que sirve de auditoría y de medición.
- **No hay deduplicación entre organizaciones clientes.** Dentro de una misma organización sí
  puede haberla, sujeta a la vigencia que configure el cliente.
- El catálogo no se limita a las listas internacionales más conocidas: la §12.1 exige que sea
  configurable y amplio.

## Fuera de alcance

- Las consultas en sí → `HU-024` y `HU-025`.
- El control de cupo y el bloqueo por cuota agotada → Fase C.
- La negociación comercial con proveedores y sus tarifas → `PA-005`, `PA-012`.
- La interfaz de administración del catálogo → Fase 5. Aquí se carga por operación de sistema.
- Auto-hospedar un motor de listas propio, descartado en `ADR-0001` para esta etapa.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `fuente.organization_id` | Sí | Organización cliente existente | No |
| `fuente.version_configuracion_id` | Sí | Versión existente (`HU-004`) | No |
| `fuente.nombre` | Sí | Único dentro de la versión | No |
| `fuente.proveedor` | Sí | Texto no vacío | No |
| `fuente.tipo` | Sí | `registro_empresarial` \| `identidad` \| `transito` \| `autoridad` \| `lista_restrictiva` \| `pep` \| `sanciones` \| `antecedentes` \| `medios` \| `comercial` | No |
| `fuente.pais` y `fuente.cobertura` | Sí | Texto no vacío | No |
| `fuente.frecuencia_actualizacion` | No | Cada cuánto actualiza el proveedor su información | No |
| `fuente.condiciones_uso` | Sí | Texto no vacío: qué permite el licenciamiento | No |
| `fuente.costo_consulta` | Sí | Monto y moneda; puede ser cero | No |
| `fuente.vigencia_resultado` | No | Cuánto tiempo se considera vigente una consulta dentro de la misma organización cliente | No |
| `credencial.organization_id` | Sí | Credenciales propias por organización cliente (§31) | Sí |

## Trazabilidad

- Épica: `EP-003`
- Capacidad: `CAP-03`
- Documento del cliente: Fase 11, §12.1, §31, §34, §44 pregunta F
- Decisiones: `ADR-0004` (las fuentes son configuración), `ADR-0001` (proveedor consolidado, no
  auto-hospedar, evento inmutable con costo)
- Integraciones: `07-integraciones/README.md`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-005` — bloqueante: qué fuentes se consultan y con qué proveedor.
  `PA-012` — cuánto cobra por consulta un proveedor local colombiano. **Queda en `borrador`.**
- **Supuestos:** `SUP-003` (contexto colombiano).
- **Depende de:** `HU-004` (versiones de configuración), `HU-002`.
- **Habilita a:** `HU-024`, `HU-025`, y la medición de consumo de la Fase C.
- **Riesgo:** el costo variable del producto nace aquí y no se puede acotar mientras `PA-005`,
  `PA-012` y `PA-014` sigan abiertas. Es el número que decide si el modelo comercial de `EP-007`
  tiene margen.
