---
id: HU-037
titulo: Administrar fuentes y aviso de privacidad
estado: borrador
epica: EP-005
prioridad: Should
actualizado: 2026-08-27
---

# HU-037 — Administrar fuentes y aviso de privacidad

## Historia

**Como** Administrador de una organización cliente
**quiero** decidir yo qué fuentes se consultan y con qué aviso de privacidad se le pide la
información a la contraparte
**para** controlar lo que gasto en consultas y lo que declaro jurídicamente ante los titulares de
los datos.

## Contexto

Dos configuraciones distintas que comparten pantalla porque comparten a su responsable y su
momento de uso: **el catálogo de fuentes** (`HU-023`) y **el aviso de privacidad con su base
jurídica** (`HU-011`).

La primera tiene consecuencia económica directa: cada fuente activada es costo por consulta, y la
§31 impide compartir esas consultas entre clientes del SaaS. La segunda tiene consecuencia
jurídica: el cliente es el responsable del tratamiento (§40) y de ahí que sea él quien redacta y
publica su aviso.

Las dos son configuración versionada. Cambiarlas es publicar, no editar.

## Criterios de aceptación

```gherkin
Escenario: Activar una fuente y ver su costo
  Dado un borrador de configuración
  Cuando el Administrador activa una fuente del catálogo disponible
  Entonces ve su costo por consulta, su cobertura y sus condiciones de uso antes de confirmar
  Y la fuente queda activa solo tras publicar el borrador
```

```gherkin
Escenario: Asociar una fuente a un requisito
  Dado una fuente activa y un requisito de la matriz
  Cuando el Administrador asocia esa fuente a la verificación de ese requisito
  Entonces las verificaciones posteriores usan esa fuente para ese campo
  Y la asociación queda registrada en la configuración
```

```gherkin
Escenario: Desactivar una fuente no borra lo ya consultado
  Dado una fuente con consultas ya ejecutadas
  Cuando el Administrador la desactiva y publica
  Entonces no se ejecutan consultas nuevas contra ella
  Y las evidencias obtenidas antes se conservan íntegras y legibles
```

```gherkin
Escenario: Publicar una versión nueva del aviso de privacidad
  Dado un borrador con un aviso de privacidad modificado
  Cuando el Oficial de Cumplimiento lo publica
  Entonces las contrapartes que entren desde ese momento ven la versión nueva
  Y quienes ya aceptaron conservan la evidencia de la versión que aceptaron
```

```gherkin
Escenario: Declarar las finalidades y la base jurídica
  Dado un borrador de aviso de privacidad
  Cuando el Administrador declara las finalidades del tratamiento y si cada una exige autorización explícita
  Entonces esa declaración queda registrada como parte de la versión
  Y el sistema no evalúa si la base jurídica es suficiente
```

```gherkin
Escenario: Un aviso incompleto no se publica
  Dado un borrador de aviso sin responsable del tratamiento o sin canales para ejercer derechos
  Cuando se intenta publicar
  Entonces la publicación es rechazada indicando qué falta
```

```gherkin
Escenario: Las credenciales de fuentes son propias de cada organización cliente
  Dado credenciales de acceso a una fuente configuradas por "Alfa Ficticia S.A.S."
  Cuando "Beta Ficticia S.A.S." consulta esa misma fuente
  Entonces usa sus propias credenciales
  Y ninguna organización cliente puede ver ni usar las credenciales de la otra
```

```gherkin
Escenario: Aislamiento entre organizaciones sobre fuentes y avisos
  Dado un administrador de "Alfa Ficticia S.A.S."
  Cuando administra fuentes y avisos con su contexto de usuario propagado
  Entonces solo ve y modifica los de "Alfa Ficticia S.A.S."
```

## Reglas de negocio

- Fuentes y aviso son **contenido de una versión de configuración**: se publican, no se editan
  (`ADR-0004`).
- Antes de activar una fuente se muestran su costo por consulta, su cobertura y sus condiciones de
  uso. Activar una fuente es una decisión económica y debe verse como tal.
- Desactivar una fuente detiene las consultas nuevas y **no afecta** a la evidencia ya obtenida.
- Las **credenciales son por organización cliente** (§31) y se almacenan cifradas; nunca llegan al
  navegador ni se comparten.
- El aviso de privacidad no se publica incompleto: exige finalidades, responsable, encargado,
  derechos del titular y canales para ejercerlos (§5).
- La plataforma **no juzga la suficiencia** de la base jurídica declarada; registra qué declaró el
  cliente (§40).
- Publicar el aviso corresponde a quien tenga el permiso; según la matriz base, es competencia del
  Oficial de Cumplimiento por su implicación jurídica.

## Fuera de alcance

- La ejecución de consultas → `HU-024` y `HU-025`.
- La negociación con proveedores y sus tarifas → `PA-005`, `PA-012`.
- La redacción del aviso: la escribe el cliente con su asesoría jurídica.
- El control de cupo por fuente → Fase C.
- La incorporación de un proveedor nuevo al producto: es desarrollo, no configuración.

## Datos y validaciones

| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| `fuente.activa` | Sí | Verdadero o falso dentro de la versión | No |
| `fuente.costo_consulta` | Sí | Visible antes de activar | No |
| `fuente.condiciones_uso` | Sí | Texto no vacío | No |
| `asociacion_fuente_requisito` | No | La fuente debe estar activa en la misma versión | No |
| `credencial.organization_id` | Sí | Propias por organización cliente; almacenadas cifradas | Sí |
| `aviso.finalidades` | Sí | Al menos una | No |
| `aviso.responsable` / `encargado` | Sí | Texto no vacío | No |
| `aviso.canales_derechos` | Sí | Texto no vacío | No |
| `aviso.exige_autorizacion` | Sí | Por finalidad | No |

## Trazabilidad

- Épica: `EP-005`
- Capacidad: `CAP-05`
- Documento del cliente: Fase 5, Fase 11, §31, §40
- Decisiones: `ADR-0004`
- Historias: pone interfaz a `HU-023` y `HU-011`

## Dependencias y riesgos

- **Preguntas abiertas:** `PA-005` (qué fuentes existen para activar), `PA-012` (su costo),
  `PA-021` (qué proveedores de IA se declaran en el aviso). **Queda en `borrador`.**
- **Supuestos:** `SUP-004`, `SUP-008`.
- **Depende de:** `HU-023`, `HU-011`, `HU-034`, `HU-003`.
- **Habilita a:** que el cliente controle su costo variable y su exposición jurídica sin
  intermediarios.
