---
id: PROD-roadmap
estado: propuesto
actualizado: 2026-09-05
---

# Roadmap por fases

> **Resuelve `PA-023`.** El cliente confirma que el alcance de la §35 es el MVP, y acuerda
> **entregarlo por fases, cubriendo un flujo completo cada vez.** Este documento define cómo
> se cortan esas fases.
>
> **Actualización 2026-09-05.** El cliente propuso además una escalera de MVPs (MVP1 a MVP5)
> al cerrar `PA-023`. Coincide con este faseo; la equivalencia está en la sección
> "Escalera de MVPs" más abajo. Lo que **sí cambia** es la posición de la interfaz de
> configuración: `PA-017` la mete dentro del MVP, y este documento la tenía en la Fase 5.
> Ver "Corrección: la configuración se adelanta a medias".

## El principio: se corta en vertical, no en horizontal

Hay dos formas de partir este proyecto, y solo una funciona.

**Corte horizontal** (por capa): primero toda la configuración, después todos los documentos,
después todo el screening. Suena ordenado y es una trampa: **nada es usable hasta el final**.
El cliente no puede probar nada, no aparece retroalimentación, y el riesgo se acumula entero
hasta el último mes.

**Corte vertical** (por recorrido): cada fase entrega un camino completo de punta a punta —
más limitado que el anterior, pero **entero y usable**. El cliente lo pone en producción, lo
usa con contrapartes reales, y lo que aprende ahí corrige la fase siguiente.

Es lo que pidió Juan David y es lo correcto. Con una condición que no se puede negociar.

## La condición: los cimientos no se fasean

Cuatro cosas tienen que estar completas **antes** de la primera fase entregable, porque
añadirlas después no es ampliar sino reescribir:

1. **Aislamiento multi-tenant con RLS** y sus pruebas.
2. **El modelo de procedencia** (`ADR-0005`). Si el primer expediente nace guardando valores
   planos, la conciliación y la auditoría no se pueden montar encima después.
3. **El versionamiento de la configuración** (`ADR-0004`). Un expediente sin la versión con la
   que fue evaluado es un expediente inauditable, y eso no se arregla hacia atrás.
4. **La bitácora inmutable**, desde la primera tabla.

Esta es la Fase 0. **No se demuestra y no entrega valor visible**, y aun así es la que
determina si el resto del proyecto es posible. Conviene decirlo así de claro cuando se
presente el plan, para que ese mes no se lea como un mes perdido.

## Las fases

| # | Fase | Qué entrega | ¿Usable? | Sem. |
|---|---|---|---|---:|
| **0** | **Cimientos** | Multi-tenant, RLS, afirmaciones, versionamiento, bitácora, cuentas y roles | No | 5 |
| **1** | **Un expediente completo, a mano** | Crear solicitud → la contraparte entra por enlace → diligencia el formulario → sube documentos → el oficial revisa y decide → expediente cerrado y auditable | **Sí** | 6 |
| **2** | **El expediente se llena solo** | Extracción con IA, conciliación declarado vs. extraído, vigencias y estados de documento, firma nivel 1 | **Sí** | 5 |
| **3** | **El expediente se verifica** | Catálogo de fuentes, verificación externa, screening, matching, alertas y casos | **Sí** | 6 |
| **4** | **El expediente se califica** | Motor de relaciones y beneficiario final, motor de riesgo configurable, debida diligencia intensificada | **Sí** | 6 |
| **5** | **El cliente se autogestiona** | Interfaz de administración: estándares, tipos de contraparte, matriz de requisitos, metodología de riesgo | **Sí** | 6 |
| **6** | **El expediente vive** | Monitoreo continuo, renovaciones, vencimientos, panel del Oficial de Cumplimiento | **Sí** | 5 |
| **C** | **Capa comercial** | Cuotas, medición de consumo, cobro y facturación DIAN (`ADR-0002`) | — | 5 |
| | **Reserva** (repartida) | | | 5 |
| | | | | **49 ≈ 11 meses** |

## Escalera de MVPs (`PA-023`)

El cliente planteó el corte en estos términos. No es un plan distinto: es el mismo, agrupado.

| MVP del cliente | Qué incluye | Fases de este roadmap |
|---|---|---|
| **MVP1** | Onboarding, formularios, documentos, OCR, expediente | 0 + 1 + 2 |
| **MVP2** | Screening, riesgo, debida diligencia intensificada | 3 + 4 |
| **MVP3** | Monitoreo continuo | 6 |
| **MVP4** | Integraciones (API de salida hacia otros sistemas del cliente) | Nuevo — ver abajo |
| **MVP5** | Inteligencia avanzada | Después del MVP |

Dos observaciones sobre el encaje:

- **La Fase 5 (autogestión) no aparece en la escalera del cliente.** No es un olvido: para él,
  la configuración es algo que se entrega configurado. Ver la corrección de más abajo.
- **El MVP4 es alcance nuevo.** Al cerrar `PA-010` el cliente confirmó que la plataforma no
  reemplaza un sistema existente sino que **alimenta otros**: hace falta una salida por API o
  por archivo estructurado (JSON/TXT configurable) para llevar la información del expediente a
  otros procesos. No estaba en este roadmap y hay que ubicarlo — encaja después de la Fase 4,
  antes o en paralelo con la 6.

## Corrección: la configuración se adelanta a medias

La versión anterior de este documento decía que **la Fase 5 va tarde a propósito** porque
hasta entonces cargamos la configuración nosotros. La respuesta a `PA-017` obliga a matizarlo:
el cliente **sí** quiere configurar, sobre plantillas base que entreguemos nosotros, y
considera eso parte del MVP.

El ajuste no es mover la Fase 5 entera hacia adelante. Es partirla en dos:

| Qué | Cuándo | Por qué |
|---|---|---|
| **Plantillas base por régimen y sector** + carga inicial como servicio de implementación | Fase 1 | Es la forma en que el primer cliente arranca, y se vende como servicio |
| **Asistente de configuración, duplicar plantilla, versionar y simulador** ("¿qué le voy a pedir a esta contraparte?") | Fase 5, sin cambios | Es la interfaz completa, y sigue siendo la palanca de ahorro más grande del plan |

Lo que se adelanta es el **contenido** (las plantillas), no la **interfaz**. Con un solo
cliente ancla, configurarlo a mano sobre plantillas ya hechas sigue siendo más rápido que
construir la pantalla. La Fase 5 sigue siendo la que convierte esto en un producto vendible a
un segundo cliente.

### Por qué ese orden

**La Fase 1 es la más importante y la que suele saltarse.** Un expediente completo sin IA, sin
screening y sin motor de riesgo parece pobre — y sin embargo, con solo eso el cliente ya
reemplaza su Excel y su cadena de correos, y empieza a tener trazabilidad real. Es lo mínimo
que se puede poner en producción, y es donde se descubre si el modelo de datos aguanta, antes
de haber construido seis módulos encima.

**La Fase 5 va tarde a propósito**, con el matiz de la sección anterior. Hasta ahí, **la
configuración la cargamos nosotros** —la matriz de requisitos, los tipos de contraparte, la
metodología— sobre plantillas base, directamente en la base de datos. El motor configurable
existe desde la Fase 0; lo que no existe es la interfaz para que el cliente lo maneje solo. Es
la palanca de ahorro más grande del plan y encaja de forma natural con tener un solo cliente
ancla. La Fase 5 es, en realidad, **la fase que convierte esto en un producto vendible a un
segundo cliente**.

**La capa comercial es móvil, y ahora pesa menos.** Va donde aparezca el primer cliente que
pague. Una respuesta la aligera: el cobro es **por facturación, sin débito automático**
(`PA-016`), lo que elimina la tokenización de tarjetas y el problema del monto variable.

> **Actualización 2026-09-07.** `PA-038` decía "alta manual al principio, autoservicio
> después" — Camilo decidió adelantar el **mecanismo** de registro público al MVP actual
> (`HU-060`, `EP-001`). La capa comercial que lo rodea (planes, cobro, KYB) **no** se adelanta:
> sigue exactamente donde estaba, bloqueada por `PA-043`. Una organización que se registra
> sola hoy no tiene plan ni cupo asignado hasta que esa capa exista.

Lo que no se puede recortar es el **medidor de consumo**, que va en la Fase 0
junto con la bitácora porque es la misma tabla.

## Cómo se mide si una fase está terminada

El propio documento del cliente da el criterio, en su §44: dieciséis preguntas que la
plataforma debe poder responder ante una auditoría. **Cada fase se define por cuáles de esas
preguntas deja contestadas.** Es mejor criterio que una lista de pantallas, porque mide valor
de cumplimiento y no volumen de trabajo.

| Fase | Preguntas de la §44 que quedan cubiertas |
|---|---|
| 1 | A quién · qué información entregó · qué documentos presentó · quién decidió · por qué · qué condiciones |
| 2 | + qué información fue extraída automáticamente |
| 3 | + qué información fue verificada · qué fuentes se consultaron · qué alertas aparecieron · quién las analizó |
| 4 | + qué metodología de riesgo se aplicó · qué nivel resultó · si hubo debida diligencia intensificada |
| 6 | + cuándo debe actualizarse · qué ocurrió durante el monitoreo |

Al cerrar la Fase 6 están las dieciséis. Ese es el producto terminado según el criterio del
propio cliente.

## Reglas de operación de las fases

1. **Cada fase se despliega a producción y se usa.** No se acumulan entregas. Una fase que no
   se puso en manos de alguien no está terminada.
2. **Cada fase arranca con una demostración de la anterior** con un caso real, no con datos de
   prueba. Es donde aparecen los requisitos que nadie había escrito.
3. **Ninguna fase deja el sistema a medias.** Si algo no cabe, se recorta alcance funcional —
   nunca trazabilidad, aislamiento ni evidencia.
4. **Lo que se aprende en una fase corrige la siguiente**, y eso se refleja aquí. Este
   documento se actualiza al final de cada fase; las estimaciones posteriores a la fase en
   curso son orientativas.

## Advertencia honesta sobre el calendario

**Fasear no reduce el trabajo total.** Son once meses en fases o son once meses de corrido; lo
que cambia es cuándo llega el valor, cuándo se puede empezar a cobrar y cuánto riesgo se
acumula antes de la primera prueba real. El beneficio es grande, pero no es un descuento.

Lo que sí reduce el total es descubrir, al usar las fases tempranas, que algo de lo planeado
no hacía falta. Eso pasa casi siempre — pero no se puede presupuestar por adelantado.

Las palancas de compresión, actualizadas con las respuestas del cliente:

- **Estándares y tipos de contraparte** — el primer despliegue ya está acotado a dos
  estándares (SARLAFT y PTEE) y siete tipos (`PA-018`). Palanca parcialmente usada.
- **Firma y segundo factor** — el OTP por SMS queda fuera y la firma digital certificada pasa
  a módulo opcional posterior (`PA-019`, `PA-020`, `ADR-0010`). Palanca **ya aplicada**.
- **Débito automático** — fuera del MVP (`PA-016`). Palanca **ya aplicada**. El registro
  público como *mecanismo* ya no es una palanca de compresión: se adelantó al MVP (`HU-060`,
  2026-09-07) — la capa comercial que lo acompañaría sigue siendo la palanca, no el alta en sí.
- **Un segundo desarrollador** — sigue disponible, sin usar.

## Orden de construcción dentro de cada fase

Vale para todas, y es la §45 del cliente:

> **"No desarrollar primero la pantalla. Desarrollar primero el modelo de cumplimiento."**

## Qué sigue

1. **Validar este faseo con Juan David** — sobre todo que la Fase 1, sin IA ni screening, le
   sirve para empezar a usarla de verdad. De eso depende todo el plan. Y validar el encaje con
   su escalera MVP1–MVP5, incluido dónde entra el MVP4 (salida por API).
2. ~~Cerrar `PA-012` y `PA-016`~~ — `PA-016` cerrada (no hay débito automático). `PA-012`
   respondida con una estimación; la cotización formal es **`PA-040`**, y sigue siendo la
   gestión más urgente.
3. **Arrancar la Fase 0**, que no depende de ninguna pregunta abierta.
4. ~~Escribir `EP-000` y `EP-001`~~ — hechas. Sigue: revisar `HU-001` a `HU-016` contra las
   respuestas del cliente y subir a `en-revision` las que ya no dependan de una `PA-xxx`.
5. ~~Ubicar el MVP4 (salida por API / archivo estructurado) en el plan y escribir su épica~~ —
   hecha: `EP-008`. Sus historias siguen sin escribir porque dependen de `PA-046` a `PA-048`.
