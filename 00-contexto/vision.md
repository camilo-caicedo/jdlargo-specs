---
id: CTX-vision
estado: propuesto
actualizado: 2026-09-05
---

# Visión del producto

> Actualizado con las respuestas del cliente a `PA-001`, `PA-003`, `PA-004`, `PA-010`,
> `PA-022` y `PA-023` (jornada de cierre de preguntas abiertas).

## Problema

Una empresa obligada a SARLAFT, SAGRILAFT o PTEE tiene que conocer a cada contraparte antes
de vincularla y seguir conociéndola después: pedirle datos y documentos, contrastarlos contra
fuentes externas, calificar el riesgo, decidir con un responsable identificado y poder
demostrarlo años después ante un supervisor.

Hoy ese proceso se sostiene en correos, formatos en Excel y consultas manuales fuente por
fuente. El costo no está en la consulta: está en **reconstruir después qué se hizo, con qué
información, bajo qué regla y quién decidió**. Cuando llega la auditoría, esa reconstrucción
casi nunca existe.

## Propuesta de valor

**Automatizamos y trazamos el proceso de debida diligencia.** La plataforma recolecta,
extrae, verifica, evalúa y deja registro; la decisión y la responsabilidad siguen siendo del
cliente.

No es una certificadora ni un sustituto del criterio del Oficial de Cumplimiento: es la
herramienta que le entrega el expediente armado, la evidencia guardada y la explicación de
por qué el sistema pidió lo que pidió (`PA-003`, `PA-022`).

## Objetivos

1. **Reducir la carga administrativa** del ciclo de vinculación: solicitud, recolección,
   documentos, verificación y decisión en un solo recorrido.
2. **Producir evidencia reconstruible por construcción**, no un reporte fabricado aparte:
   expediente electrónico con procedencia del dato, versión de configuración congelada y
   bitácora (`ADR-0004`, `ADR-0005`).
3. **Dejar el marco de cumplimiento en manos del cliente**: estándares activables, matriz de
   requisitos y metodología de riesgo configurables sobre plantillas base (`PA-006`, `PA-017`).
4. **Mantener vivo el expediente** después de la vinculación, con monitoreo continuo por
   riesgo y por evento (`PA-011`, `PA-035`).
5. **Alimentar otros sistemas del cliente** con la información del expediente, por API o por
   archivo estructurado (`PA-010`).

## No-objetivos (fuera de alcance)

- **No certifica.** La plataforma no emite una "certificación de cumplimiento" ni declara a
  una empresa "certificada". Emite un **Informe de Debida Diligencia** y una **constancia del
  proceso ejecutado**, con sus disclaimers (`PA-003`, `PA-022`).
- **No decide.** El sistema propone, calcula y alerta; aprueba una persona identificada.
- **No evalúa a la propia empresa contratante** como parte del producto. El cliente pasa por
  un onboarding/KYB comercial para contratar el SaaS, que es un flujo distinto del de debida
  diligencia que él aplica a sus terceros (`PA-004`).
- **No genera reportes regulatorios (ROS y similares) como consecuencia automática** de una
  debida diligencia. Una consulta de contraparte no equivale a un reporte a la autoridad
  (`PA-007`).
- **No impone una metodología de riesgo única.** Trae una de referencia; la política es del
  cliente (`PA-006`).
- **No reemplaza un sistema existente** en el primer cliente: lo alimenta. La migración se
  soporta con plantillas controladas, pero no bloquea el MVP (`PA-010`).

## Métricas de éxito

Se miden sobre el panel del Oficial de Cumplimiento (`HU-043`, `PA-039`):

| Métrica | Qué demuestra |
|---|---|
| Tiempo desde la solicitud hasta la decisión | Reducción de carga administrativa (objetivo 1) |
| Tiempo desde la alerta hasta su resolución | Capacidad real de operar el riesgo |
| % de expedientes completos sin intervención manual | Efecto de la automatización y del OCR/IA |
| % de las 16 preguntas de la §44 respondibles sobre un expediente al azar | Trazabilidad (objetivo 2) |
| Falsos positivos de matching por cada 100 coincidencias | Calidad del screening (`PA-033`) |

Los valores objetivo se fijan al cierre de la Fase 1, con datos reales del cliente ancla. No
se inventan hoy.

## Alcance regulatorio

Cierra `PA-001`.

| Marco | ¿En alcance? | Notas |
|-------|--------------|-------|
| **SARLAFT** | Sí — núcleo del MVP | El sector transporte tiene marco SARLAFT propio y actualizado (Supertransporte) `(por validar)` |
| **SAGRILAFT** | Sí — núcleo del MVP | Vigilancia de la Superintendencia de Sociedades `(por validar)` |
| **PTEE** | Sí — **marco complementario** | Entra al MVP, pero no se modela como si fuera idéntico a los otros dos: es anticorrupción/antisoborno, con requisitos propios |
| Otros estándares (ISO 37001, SARO, DD de terceros…) | Después del MVP | El motor debe admitirlos sin rediseño |

La consecuencia de diseño es una capa genérica, no tres módulos rígidos:

```
Estándar → versión → requisito → regla → formulario → documento → control
```

Cada estándar se puede **activar o desactivar** por organización cliente. En el primer
cliente se despliegan dos: SARLAFT y PTEE (`PA-018`).

> ⚠️ Las referencias normativas concretas que el cliente aporta (circular de Supersociedades
> vigente desde julio de 2026, marco de Supertransporte) **no están verificadas contra fuente
> oficial**. Se usan como contexto de diseño, nunca como hecho legal — ver `SUP-008`.

## Posicionamiento

El nombre inicial del proyecto hablaba de "validación, verificación y **certificación** de
entidades". Se retira "certificación" del posicionamiento principal (`PA-022`):

- **Sí:** "Automatización de Debida Diligencia" / *Compliance Due Diligence Platform*.
- **No:** "certificación de cumplimiento", "empresa certificada", o cualquier fórmula que
  cree una falsa garantía frente a un supervisor.
- "Certificado" solo puede existir como **nombre de un reporte**, nunca como promesa.

Ver `ADR-0006`.

## Preguntas abiertas relacionadas

Ninguna bloqueante. Quedan derivadas: `PA-043` (precios definitivos) y `PA-044` (política de
retención definitiva).
