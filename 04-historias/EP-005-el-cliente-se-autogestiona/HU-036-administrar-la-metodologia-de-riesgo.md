---
id: HU-036
titulo: Administrar la metodología de riesgo
estado: borrador
epica: EP-005
prioridad: Must
actualizado: 2026-08-27
---

# HU-036 — Administrar la metodología de riesgo

## Historia

**Como** Oficial de Cumplimiento
**quiero** ajustar yo mismo los factores, los pesos y los umbrales de mi metodología
**para** responder por ella de verdad, que es lo que me exige mi cargo, en vez de responder por un
cálculo que configuró otro.

## Contexto

La §40 reparte la responsabilidad sin ambigüedad: **el cliente responde por su metodología y por
cómo clasifica el riesgo**. Difícil sostener eso si la metodología solo se puede cambiar pidiendo
un favor al proveedor del software.

`HU-031` construyó la metodología como configuración versionada; esta historia la pone en manos
de quien responde por ella. La dificultad no es técnica: es que un cambio de peso o de umbral
tiene efectos que no se ven hasta que se aplican a expedientes reales, y quien lo hace necesita
entenderlos **antes** de publicar.

## Criterios de aceptación

```gherkin
Escenario: Ajustar el peso de un factor
  Dado un borrador de configuración con una metodología
  Cuando el Oficial de Cumplimiento cambia la ponderación de un factor
  Entonces el borrador refleja el cambio
  Y la metodología vigente no se altera mientras no se publique
```

```gherkin
Escenario: Añadir un factor de riesgo
  Dado un borrador de metodología
  Cuando el Oficial de Cumplimiento añade un factor con su ponderación y su escala
  Entonces el factor queda disponible para la evaluación
  Y el sistema verifica que el factor se pueda alimentar con datos que el expediente recoge
```

```gherkin
Escenario: Un factor sin dato que lo alimente se señala
  Dado un factor de riesgo que depende de un campo que la matriz de requisitos no pide
  Cuando se valida el borrador
  Entonces el sistema advierte que ese factor nunca tendrá dato
  Y no permite publicar hasta que se corrija o se acepte explícitamente
```

```gherkin
Escenario: Cambiar un umbral y ver el efecto
  Dado una metodología con umbrales definidos
  Cuando el Oficial de Cumplimiento modifica un umbral
  Entonces se le muestra cómo se redistribuirían los niveles con ese cambio
  Y se le indica sobre qué expedientes recientes habría cambiado el resultado
```

```gherkin
Escenario: Definir una regla de escalamiento
  Dado un borrador de metodología
  Cuando el Oficial de Cumplimiento define que cierta combinación exige debida diligencia intensificada
  Entonces la regla queda registrada usando el conjunto cerrado de condiciones y acciones
  Y se elige de listas, sin escribir expresiones
```

```gherkin
Escenario: Una escala incoherente no se publica
  Dado una metodología cuyos umbrales dejan un rango sin nivel o se superponen
  Cuando se intenta publicar
  Entonces la publicación es rechazada indicando el rango afectado
```

```gherkin
Escenario: Publicar exige el permiso de modificar la metodología
  Dado un usuario cuyo rol no incluye el permiso de modificar la metodología
  Cuando intenta publicar un cambio en ella
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Los expedientes ya evaluados no cambian
  Dado expedientes evaluados con la metodología vigente
  Cuando se publica una metodología nueva
  Entonces esos expedientes conservan su evaluación y la versión con la que se hizo
  Y las evaluaciones posteriores usan la nueva
```

## Reglas de negocio

- Los cambios se hacen sobre un **borrador** y solo tienen efecto al publicar (`HU-034`).
- El sistema valida que cada factor **se pueda alimentar** con datos que el expediente
  efectivamente recoge. Un factor sin fuente de datos es una ponderación que nunca se aplica.
- Los umbrales deben cubrir toda la escala, sin huecos ni superposiciones.
- Las reglas de escalamiento se componen con el conjunto cerrado de `ADR-0004`; el usuario elige,
  no escribe.
- Modificar la metodología requiere el permiso correspondiente, que según la matriz base es del
  Oficial de Cumplimiento (§30).
- Publicar una metodología nueva **no recalcula** las evaluaciones anteriores: cada una conserva
  su versión (`ADR-0005`).
- El sistema muestra el efecto probable del cambio antes de publicar, sobre expedientes reales
  recientes.

## Fuera de alcance

- El cálculo en sí → `HU-032`.
- La simulación completa contra un expediente de ensayo → `HU-039`.
- El recálculo masivo de expedientes ya evaluados: no está definido y tiene implicaciones
  regulatorias que no nos corresponde suponer.
- Sugerencias automáticas de metodología o comparación con otros clientes → fase posterior (§36).
- La validación regulatoria de la metodología: es del cliente (§40).

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `factor.clave` | Sí | Único dentro de la metodología | No |
| `factor.origen_dato` | Sí | Campo o resultado que efectivamente existe en el expediente | No |
| `factor.ponderacion` | Sí | Numérica y coherente con la escala | No |
| `escala.niveles` | Sí | Al menos dos, sin huecos ni superposiciones | No |
| `regla_escalamiento.condicion` / `accion` | Sí | Solo del conjunto cerrado | No |
| Permiso de publicar metodología | Sí | Según la matriz de permisos vigente | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: Fase 14, §30, §40, §41
- Decisiones: `ADR-0004`, `ADR-0005`
- Historias: pone interfaz a `HU-031`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-017`, `PA-034` (si la clasificación exige segunda aprobación, la
  metodología debe poder configurarlo). **Queda en `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-031`, `HU-034`, `HU-003`.
- **Habilita a:** que el cliente responda de verdad por su metodología, que es lo que la §40 le
  atribuye.
- **Riesgo:** un cambio de peso mal entendido puede reclasificar a media cartera sin que nadie lo
  note hasta la siguiente auditoría. Por eso el criterio de "ver el efecto antes de publicar" no
  es una comodidad: es el control que evita ese fallo.
