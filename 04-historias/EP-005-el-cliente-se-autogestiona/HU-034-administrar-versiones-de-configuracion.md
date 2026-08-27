---
id: HU-034
titulo: Administrar versiones de configuración
estado: borrador
epica: EP-005
prioridad: Must
actualizado: 2026-08-27
---

# HU-034 — Administrar versiones de configuración

## Historia

**Como** Oficial de Cumplimiento
**quiero** preparar los cambios de configuración en un borrador, ver en qué se diferencian de lo
vigente y publicarlos cuando esté seguro
**para** cambiar lo que mi empresa exige sin llamar a nadie, y sin miedo a romper lo que ya está
funcionando.

## Contexto

`HU-004` construyó el mecanismo: versiones inmutables, puntero de configuración vigente, nada de
edición en sitio. Esta historia le pone la interfaz, y con ella aparecen dos necesidades que en
la carga manual no existían: **comparar** antes de publicar y **entender qué va a cambiar**.

`ADR-0004` lo anticipó: la interfaz de administración tiene que ser usable por un Oficial de
Cumplimiento, no por un desarrollador. Eso significa que la comparación no puede ser un volcado
de filas: tiene que decir "a partir de ahora se exigirá el documento X a los proveedores".

## Criterios de aceptación

```gherkin
Escenario: Crear un borrador a partir de la versión vigente
  Dado una organización cliente con una versión de configuración vigente
  Cuando el Oficial de Cumplimiento crea un borrador nuevo
  Entonces el borrador parte de una copia del contenido vigente
  Y la versión vigente no se modifica en ningún momento
  Y el borrador es editable libremente
```

```gherkin
Escenario: Comparar el borrador con lo vigente
  Dado un borrador con cambios respecto a la versión vigente
  Cuando el Oficial de Cumplimiento pide la comparación
  Entonces ve qué se añade, qué se quita y qué cambia
  Y lo ve descrito en términos de su negocio, no como filas de una tabla
```

```gherkin
Escenario: Publicar exige motivo y permiso
  Dado un borrador listo
  Cuando el Oficial de Cumplimiento lo publica indicando el motivo del cambio
  Entonces la versión queda publicada e inmutable
  Y pasa a ser la configuración vigente
  Y queda registrado quién publicó, cuándo y por qué
```

```gherkin
Escenario: Publicar sin permiso es rechazado
  Dado un usuario cuyo rol vigente no incluye el permiso de publicar configuración
  Cuando intenta publicar un borrador
  Entonces la acción es rechazada
  Y el intento queda registrado en la bitácora
```

```gherkin
Escenario: Ver el historial de versiones
  Dado una organización cliente con varias versiones publicadas
  Cuando el Auditor consulta el historial
  Entonces ve cada versión con su número, su autor, su fecha de publicación, su motivo y su periodo de vigencia
  Y puede abrir el contenido de cualquiera de ellas tal como estaba
```

```gherkin
Escenario: Volver a una configuración anterior es publicar una versión nueva
  Dado una versión publicada que se quiere deshacer
  Cuando el Oficial de Cumplimiento decide volver al contenido anterior
  Entonces se crea un borrador con ese contenido y se publica como versión nueva
  Y ninguna versión anterior se modifica ni se elimina
  Y los expedientes evaluados con cada versión siguen citando la suya
```

```gherkin
Escenario: Un borrador incompleto no se publica
  Dado un borrador con configuración incoherente o incompleta
  Cuando se intenta publicar
  Entonces la publicación es rechazada indicando qué falta y dónde
  Y el borrador permanece editable
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre la administración
  Dado un usuario miembro únicamente de "Alfa Ficticia S.A.S."
  Cuando administra versiones de configuración con su contexto de usuario propagado
  Entonces solo ve y modifica las de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- El borrador es la **única** superficie editable. Una versión publicada nunca se edita
  (`ADR-0004`).
- Publicar exige **motivo** y el permiso correspondiente (`HU-003`).
- La comparación entre borrador y versión vigente se expresa en términos de negocio: qué se
  empieza a exigir, qué deja de exigirse, qué cambia de peso.
- Solo puede haber **un borrador activo a la vez** por organización cliente, para que no se
  publiquen cambios que se pisan entre sí.
- Deshacer un cambio es publicar una versión nueva con el contenido anterior. **No existe la
  eliminación de versiones.**
- La validación de coherencia se ejecuta antes de publicar y bloquea la publicación, no avisa
  después.
- El historial completo es consultable por el Auditor, sin capacidad de escritura.

## Fuera de alcance

- El contenido concreto que se administra → `HU-035` a `HU-038`.
- La prueba de la configuración contra un expediente de ensayo → `HU-039`.
- La aprobación por doble control antes de publicar: no hay definición del cliente.
- La programación de una publicación para una fecha futura.
- La comparación entre dos versiones antiguas cualesquiera: aquí se compara el borrador con lo
  vigente.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `version_configuracion.estado` | Sí | `borrador` \| `publicada` \| `reemplazada` | No |
| Borrador activo | Sí | Como máximo uno por organización cliente | No |
| `version_configuracion.motivo` | Sí | Texto no vacío al publicar | No |
| `version_configuracion.publicada_por` | Sí | Usuario con permiso de publicar | No |
| Validación de coherencia | Sí | Bloquea la publicación si algo falta o es incoherente | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: §41, Fase 0 del cliente (configuración normativa)
- Decisiones: `ADR-0004`
- Historias: pone interfaz a `HU-004`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-017` — quién configura en la práctica; define si esta historia es
  central o marginal. `PA-025` — qué pasa con los expedientes en curso al publicar. **Queda en
  `borrador`.**
- **Supuestos:** ninguno propio.
- **Depende de:** `HU-004`, `HU-003`.
- **Habilita a:** el resto de `EP-005`.
- **Riesgo:** una comparación que muestre filas y claves técnicas cumple el criterio de existir y
  falla el de que alguien la entienda. Si el Oficial de Cumplimiento no puede leer qué va a
  cambiar, publicará a ciegas o dejará de publicar.
