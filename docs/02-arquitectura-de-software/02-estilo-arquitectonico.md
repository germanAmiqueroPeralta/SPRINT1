# Estilo arquitectónico

Se utiliza el estilo arquitectónico: **monolito modular + capas**.

| Pregunta | Respuesta en SICNA |
|---|---|
| ¿Cómo se despliega? | **Monolito**: todo el backend es una sola aplicación (`uvicorn main:app`) con una sola base de datos. |
| ¿Cómo se divide? | **Modular**: 19 módulos independientes por dominio en `srs-back/src/modules/`. |
| ¿Cómo se organiza por dentro? | **Capas**: cada módulo tiene las mismas 4 capas (presentación, aplicación, dominio e infraestructura). |

> *Capas = organización lógica; monolito = unidad de despliegue. Ambos coexisten.*

## Diagrama

```mermaid
flowchart TB
    Front["Aplicación web · Next.js"] -->|HTTP / JSON| Main

    subgraph Monolito["Backend SICNA · un solo proceso (FastAPI)"]
        Main["main.py · rutas /api"]
        Main --> M1 & M2 & M3 & M4

        subgraph M1["módulo child"]
            direction TB
            P1[presentación] --> A1[aplicación] --> D1[dominio]
            I1[infraestructura] --> D1
        end
        subgraph M2["módulo child_control"]
            direction TB
            P2[presentación] --> A2[aplicación] --> D2[dominio]
            I2[infraestructura] --> D2
        end
        subgraph M3["módulo claim"]
            direction TB
            P3[presentación] --> A3[aplicación] --> D3[dominio]
            I3[infraestructura] --> D3
        end
        M4["… 16 módulos más<br/>con la misma estructura"]
    end

    Monolito --> BD[("PostgreSQL · una sola base de datos")]
```

## ¿Por qué este estilo y no otro?

| Estilo | ¿Se eligió? | Motivo |
|---|---|---|
| **Monolito modular** | ✅ Sí | Un solo despliegue, simple para un equipo pequeño (RC07), con módulos que mantienen el orden (DA04). |
| **Capas** | ✅ Sí | Separa responsabilidades dentro de cada módulo (ver Clean Architecture). |
| Cliente-servidor | Implícito | El frontend es el cliente y la API es el servidor, como en toda aplicación web. |
| Microservicios | ❌ No | Agrega red, varios despliegues y datos distribuidos sin necesidad en esta etapa. Gracias a los módulos, en el futuro se podría separar uno como servicio. |
| Event-Driven | ❌ No | No hay procesos asíncronos que lo justifiquen; las operaciones son peticiones y respuestas directas. |
| Serverless | ❌ No | La aplicación mantiene sesiones, auditoría y conexiones a la base de datos; un servidor tradicional es más simple. |
