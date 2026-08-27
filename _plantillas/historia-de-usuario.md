---
id: HU-xxx
titulo: <título corto en infinitivo>
estado: borrador
epica: EP-xxx
prioridad: Must | Should | Could | Won't
actualizado: AAAA-MM-DD
---

# HU-xxx — <título>

> Preferir `/user-story-writing` para generar historias. Esta plantilla es el respaldo
> para ediciones manuales y para revisar que no falte nada.

## Historia
**Como** <actor de `00-contexto/actores-y-roles.md`>
**quiero** <acción>
**para** <valor / resultado esperado>.

## Contexto
<Por qué existe esta historia. Qué requisito o norma la origina.>

## Criterios de aceptación

```gherkin
Escenario: <nombre>
  Dado <estado inicial>
  Cuando <acción>
  Entonces <resultado observable>
```

```gherkin
Escenario: <caso alterno o de error>
  Dado ...
  Cuando ...
  Entonces ...
```

## Reglas de negocio
- <regla verificable>

## Fuera de alcance
- <lo que esta historia NO hace>

## Datos y validaciones
| Campo | Obligatorio | Validación | Sensible |
|-------|-------------|------------|----------|
| | | | |

## Trazabilidad
- Requisitos: `RF-xxx`
- Flujo: `FL-xxx`
- Normativa: <marco y referencia, si aplica y está verificada>

## Dependencias y riesgos
- Preguntas abiertas: `PA-xxx`
- Supuestos: `SUP-xxx`
- Depende de: `HU-xxx`

## Notas de diseño / UX
<Opcional.>
