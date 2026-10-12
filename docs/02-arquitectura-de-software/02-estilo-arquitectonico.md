# Estilo arquitectónico

Se utiliza el estilo arquitectónico: **monolito modular + capas**.

| Pregunta | Respuesta en SICNA |
|---|---|
| ¿Cómo se despliega? | **Monolito**: todo el backend es una sola aplicación con una sola base de datos. El importador de atenciones usa el mismo código. |
| ¿Cómo se divide? | **Modular**: módulos independientes por dominio en `srs-back/src/modules/`. |
| ¿Cómo se organiza por dentro? | **Capas**: cada módulo tiene las mismas 4 capas (presentación, aplicación, dominio e infraestructura). |

> *Capas = organización lógica; monolito = unidad de despliegue. Ambos coexisten.*

## Diagrama general del estilo

```mermaid
flowchart TB
    Front["🖥️ Aplicación web · Next.js"] -->|"HTTP/JSON"| Entrada
    Cron["⏱️ Programador<br/>(cada noche)"] --> Importador

    subgraph Monolito["📦 MONOLITO · Backend SICNA (una sola base de código)"]
        direction TB
        Entrada["main.py · FastAPI<br/>AuthMiddleware · require_permission · /api"]
        Importador["Importador de atenciones<br/>(mismo código, otro proceso)"]

        subgraph Modulos["🧩 MÓDULOS (modular)"]
            direction LR
            subgraph G1["Acceso y seguridad"]
                direction TB
                m_auth[auth] ~~~ m_user[user] ~~~ m_role[role] ~~~ m_perm[permission]
                m_ses[session] ~~~ m_dev[device] ~~~ m_alert[security_alert]
            end
            subgraph G2["Territorio"]
                direction TB
                m_reg[region] ~~~ m_hf[health_facility] ~~~ m_ur[user_region]
            end
            subgraph G3["Atención del niño"]
                direction TB
                m_child[child] ~~~ m_guard[guardian] ~~~ m_cg[child_guardian]
                m_pkg[care_package] ~~~ m_srv[care_service] ~~~ m_ctrl[child_control]
            end
            subgraph G4["Reclamos y portal"]
                direction TB
                m_claim[claim] ~~~ m_ev[claim_evidence] ~~~ m_portal[portal]
            end
            subgraph G5["Integraciones"]
                direction TB
                m_id[identity] ~~~ m_sync[attention_sync]
            end
            m_audit[audit_log]
        end

        Core["🔧 core/ · núcleo compartido<br/>conexión BD · errores comunes · seguridad · archivos · auditoría"]

        Entrada --> Modulos
        Importador --> m_sync
        Modulos --> Core
    end

    Core --> BD[("🗄️ PostgreSQL<br/>una sola base de datos")]
    Core --> FS[("📁 Archivos")]
    m_id -.->|HTTPS| RENIEC["🌐 API RENIEC"]
    m_sync -.->|"solo lectura"| BDEXT[("🌐 BD externa<br/>de atenciones")]
```

## Diagrama de las capas dentro de un módulo

Todos los módulos repiten la misma estructura. Ejemplo con `child_control`:

```mermaid
flowchart LR
    subgraph modulo["Módulo child_control"]
        direction LR
        P["<b>1 · Presentación</b><br/>presentation/<br/>rutas · handlers · presenter"]
        A["<b>2 · Aplicación</b><br/>application/<br/>casos de uso · validaciones"]
        D["<b>3 · Dominio</b><br/>domain/<br/>entidad · errores · puertos"]
        I["<b>4 · Infraestructura</b><br/>infrastructure/<br/>repositorio SQL · adaptadores"]
        C["child_control_container.py<br/><i>conecta las piezas</i>"]
        P --> A --> D
        I -->|implementa los puertos| D
        C -.-> P & A & I
    end
    HTTP["Petición HTTP"] --> P
    I --> BD[("PostgreSQL")]
    I --> OTROS["Otros módulos<br/>child · care_service · …"]
```

| Capa | Responsabilidad | Ejemplo |
|---|---|---|
| **Presentación** | Recibir la petición HTTP, validar el formato y responder en JSON | `child_control_create_handler.py` |
| **Aplicación** | Coordinar el caso de uso y aplicar las reglas | `ChildControlCreateUseCase` |
| **Dominio** | Definir la entidad y las interfaces (puertos) que necesita | `ChildControl`, `ChildPort` |
| **Infraestructura** | Implementar las interfaces con una tecnología concreta | `AsyncpgChildControlRepository`, `ChildAdapter` |

## ¿Por qué este estilo?

| Estilo | Motivo de la elección |
|---|---|
| **Monolito** | Un solo despliegue y una sola base de datos: simple para un equipo pequeño (RC09) y con transacciones consistentes entre niños, controles y reclamos. |
| **Modular** | Cada módulo tiene su propio dominio y se comunica con los demás solo mediante puertos, así que un cambio en un módulo no rompe a otros (DA06). Si en el futuro un módulo crece mucho, se puede separar. |
| **Capas** | Separa la forma de recibir datos (HTTP), las reglas del negocio y la tecnología de almacenamiento dentro de cada módulo. |
