---
id: FL-xxx
titulo: <nombre del flujo>
estado: borrador
actualizado: AAAA-MM-DD
---

# FL-xxx — <nombre>

## Objetivo del actor
<Qué quiere lograr y en qué momento.>

## Actor principal
<...>  |  **Otros participantes:** <...>

## Precondiciones
- <...>

## Camino principal
1. <paso>
2. <paso>

## Caminos alternos
- **A1** <cuándo ocurre> → <qué pasa>

## Errores y excepciones
- **E1** <situación> → <manejo esperado>

## Postcondiciones
- <estado final del sistema, evidencia registrada>

## Diagrama

```mermaid
flowchart TD
    A[Inicio] --> B{Decisión}
    B -->|Sí| C[Paso]
    B -->|No| D[Alterno]
```

## Historias que lo implementan
- `HU-xxx`
