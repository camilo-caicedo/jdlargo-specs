---
id: PROD-roadmap
estado: propuesto
actualizado: 2026-08-25
---

# Roadmap por fases

> **Resuelve `PA-023`.** El cliente confirma que el alcance de la §35 es el MVP, y acuerda
> **entregarlo por fases, cubriendo un flujo completo cada vez.** Este documento define cómo
> se cortan esas fases.

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

### Por qué ese orden

**La Fase 1 es la más importante y la que suele saltarse.** Un expediente completo sin IA, sin
screening y sin motor de riesgo parece pobre — y sin embargo, con solo eso el cliente ya
reemplaza su Excel y su cadena de correos, y empieza a tener trazabilidad real. Es lo mínimo
que se puede poner en producción, y es donde se descubre si el modelo de datos aguanta, antes
de haber construido seis módulos encima.

**La Fase 5 va tarde a propósito.** Hasta ahí, **la configuración la cargamos nosotros** —la
matriz de requisitos, los tipos de contraparte, la metodología— directamente en la base de
datos. El motor configurable existe desde la Fase 0; lo que no existe es la interfaz para que
el cliente lo maneje solo. Es la palanca de ahorro más grande del plan y encaja de forma
natural con tener un solo cliente ancla: mientras haya uno, configurarlo a mano es más rápido
que construir la pantalla. La Fase 5 es, en realidad, **la fase que convierte esto en un
producto vendible a un segundo cliente**.

**La capa comercial es móvil.** Va donde aparezca el primer cliente que pague. Si Juan David
opera como cliente ancla facturado a mano, puede ir después de la Fase 4. Si se abre registro
público antes, hay que adelantarla. Es la única pieza del plan que no tiene un lugar fijo.

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

Las tres palancas del plan anterior siguen disponibles si hay que comprimir: arrancar con un
solo estándar y dos o tres tipos de contraparte (`PA-018`), aplazar firma nivel 2 y OTP por
SMS (`PA-019`), y un segundo desarrollador.

## Orden de construcción dentro de cada fase

Vale para todas, y es la §45 del cliente:

> **"No desarrollar primero la pantalla. Desarrollar primero el modelo de cumplimiento."**

## Qué sigue

1. **Validar este faseo con Juan David** — sobre todo que la Fase 1, sin IA ni screening, le
   sirve para empezar a usarla de verdad. De eso depende todo el plan.
2. Cerrar `PA-012` y `PA-016`, las dos llamadas pendientes.
3. **Arrancar la Fase 0**, que no depende de ninguna de las dos.
4. Escribir `EP-000` (cimientos) y `EP-001` (expediente a mano) con `/user-story-writing`.
