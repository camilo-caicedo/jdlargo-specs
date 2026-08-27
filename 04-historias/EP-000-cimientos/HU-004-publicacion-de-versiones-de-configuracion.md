---
id: HU-004
titulo: Publicación de versiones de configuración inmutables
estado: borrador
epica: EP-000
prioridad: Must
actualizado: 2026-08-27
---

# HU-004 — Publicación de versiones de configuración inmutables

## Historia

**Como** Oficial de Cumplimiento
**quiero** que la configuración de cumplimiento de mi organización cliente se publique como
versiones inmutables, y que la anterior quede intacta
**para** poder demostrar ante una auditoría, años después, con qué reglas exactas se evaluó
cada expediente.

## Contexto

La §41 lo pide sin rodeos: cuando cambia una norma, lo que se actualiza es **la configuración
y su versión**, no el software. Y cada expediente debe guardar con qué versión fue evaluado:
estándar, norma, versión, fecha de vigencia, metodología, versión de la matriz de requisitos,
versión de las reglas y versión del formulario.

`ADR-0004` extrae la consecuencia práctica: **no existe `UPDATE` sobre una tabla de
configuración publicada.** Solo inserción de una versión nueva y movimiento del puntero de
configuración vigente. Una configuración editable en sitio rompe la auditoría: un expediente
de hace un año deja de poder reconstruirse.

Esta historia construye el **mecanismo de versionamiento**, vacío de contenido. Qué va dentro
—matriz de requisitos, metodología de riesgo, tipos de contraparte, catálogo de fuentes— lo
llenan las fases posteriores. Aquí se garantiza que, cuando llegue ese contenido, ya no se
pueda editar por debajo.

## Criterios de aceptación

```gherkin
Escenario: Publicar la primera versión de configuración
  Dado una organización cliente sin ninguna versión de configuración publicada
  Cuando el Oficial de Cumplimiento publica una versión con su contenido de cumplimiento
  Entonces la versión queda registrada como publicada, con número, autor, momento de publicación y fecha de vigencia
  Y esa versión queda señalada como configuración vigente de esa organización cliente
  Y la publicación queda registrada en la bitácora
```

```gherkin
Escenario: Una versión publicada no se puede modificar
  Dado una versión de configuración ya publicada
  Cuando se intenta modificar o eliminar cualquiera de sus filas
  Entonces la operación es rechazada por la base de datos
  Y el contenido de la versión permanece idéntico
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Cambiar la configuración es publicar una versión nueva
  Dado una organización cliente con la versión 3 vigente
  Cuando el Oficial de Cumplimiento publica la versión 4 con un cambio en su contenido
  Entonces la versión 4 queda señalada como vigente
  Y la versión 3 sigue existiendo, completa y legible, con su contenido sin alterar
  Y se puede consultar qué cambió entre la 3 y la 4
```

```gherkin
Escenario: Un borrador de configuración no rige mientras no se publique
  Dado una versión de configuración en estado borrador
  Cuando se consulta la configuración vigente de la organización cliente
  Entonces se obtiene la última versión publicada
  Y no se obtiene el borrador
  Y el borrador sí se puede modificar libremente mientras no se publique
```

```gherkin
Escenario: Reconstruir la configuración de una fecha pasada
  Dado una organización cliente con varias versiones publicadas a lo largo del tiempo
  Cuando se pide la configuración que regía en una fecha determinada
  Entonces se obtiene la versión que estaba vigente en esa fecha
  Y se obtiene su contenido tal como estaba, no el contenido actual
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la configuración
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando consulta las versiones de configuración con su contexto de usuario propagado
  Entonces obtiene únicamente las versiones de "Alfa Ficticia S.A.S."
  Y no obtiene ninguna versión de "Beta Ficticia S.A.S."
  Y un intento de publicar una versión en "Beta Ficticia S.A.S." es rechazado por la política de la base de datos
```

```gherkin
Escenario: Publicar exige el permiso correspondiente
  Dado un usuario cuyo rol vigente no incluye el permiso de publicar configuración
  Cuando intenta publicar una versión
  Entonces la acción es rechazada
  Y la bitácora registra el intento con el usuario y la versión de configuración con la que se evaluó el permiso
```

## Reglas de negocio

- Una versión de configuración pertenece a **una** organización cliente. No hay configuración
  compartida entre clientes del SaaS.
- Estados de una versión: `borrador` (modificable) → `publicada` (inmutable) →
  `reemplazada`. Una versión publicada nunca vuelve a `borrador`.
- Publicar es la única forma de cambiar la configuración vigente. **No existe `UPDATE` sobre una
  versión publicada**, y la restricción se impone en la base de datos, no por convención.
- El puntero de configuración vigente se mueve; el contenido no. Mover el puntero es, en sí
  mismo, un hecho registrado en la bitácora.
- Cada versión guarda como mínimo lo que exige la §41: estándar, norma de referencia, número de
  versión, fecha de vigencia, y las versiones de metodología, matriz de requisitos, reglas y
  formulario que contiene.
- Publicar requiere permiso explícito (`HU-003`). Según la matriz base, lo tiene el Oficial de
  Cumplimiento.
- Una versión publicada **no se borra**, ni siquiera cuando la reemplaza otra: hay expedientes
  que la citan.

## Fuera de alcance

- **El contenido** de la configuración: matriz de requisitos, metodología de riesgo, tipos de
  contraparte, catálogo de fuentes y formularios. Llegan en sus fases; aquí solo el envase.
- El evaluador de reglas y su explicación (`ADR-0004`, regla 5). Se construye cuando exista algo
  que evaluar.
- La interfaz de administración de la configuración → Fase 5.
- El congelamiento de la versión dentro del expediente: no hay expediente todavía. Es lo primero
  que hará la Fase 1, y depende de `PA-025`.
- Comparación visual entre versiones y aprobación por doble control.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `version_configuracion.organization_id` | Sí | Organización cliente existente | No |
| `version_configuracion.numero` | Sí | Entero correlativo por organización cliente, nunca reutilizado | No |
| `version_configuracion.estado` | Sí | `borrador` \| `publicada` \| `reemplazada` | No |
| `version_configuracion.estandar` | Sí | Valor del catálogo de estándares `(por validar — PA-001)` | No |
| `version_configuracion.norma_referencia` | No | Texto; se cita solo si está verificada contra fuente oficial | No |
| `version_configuracion.fecha_vigencia_desde` | Sí | Fecha; no anterior a la de la versión previa | No |
| `version_configuracion.publicada_por` | Sí | Usuario con permiso de publicar | No |
| `version_configuracion.publicada_en` | Sí | Momento de la publicación; se escribe una sola vez | No |
| `version_configuracion.motivo` | Sí | Texto no vacío: por qué se publica esta versión | No |

## Trazabilidad

- Épica: `EP-000`
- Capacidad: `CAP-00`
- Documento del cliente: §41, §42
- Decisiones: `ADR-0004`
- Modelo: `05-datos/modelo-conceptual.md`, plano de Configuración

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-025` — bloqueante. Al publicar una versión nueva, ¿los expedientes
  en curso siguen con la anterior o migran? De la respuesta depende si el expediente congela la
  versión **al abrirse** o **al evaluarse**, y son dos modelos distintos. **No pasa de
  `borrador` hasta que se responda.** `PA-017` y `PA-018` afectan al esfuerzo de carga inicial,
  no al modelo.
- **Supuestos:** `SUP-001` (alcance SARLAFT / SAGRILAFT / PTEE), `SUP-008` (las normas citadas
  por el cliente no están verificadas: el campo de norma de referencia se llena, no se afirma).
- **Depende de:** `HU-002` (aislamiento), `HU-006` (la publicación se registra en la bitácora).
- **Habilita a:** `HU-003` y toda la configuración de cumplimiento de las fases 2 a 6.
- **Riesgo:** es la historia que menos se nota si sale mal y más cuesta corregir. Una
  configuración que se pudo editar en sitio, aunque fuera una vez, deja expedientes que ya no
  se pueden reconstruir, y eso no se arregla hacia atrás.
