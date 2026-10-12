# Arquitectura inicial del sistema

## Diagrama de arquitectura

```mermaid
flowchart TB
    subgraph USUARIOS["👥 USUARIOS"]
        direction LR
        AG["Administrador general<br/><i>rol admin</i>"]
        AL["Administrador limitado<br/><i>rol health_staff</i>"]
        PA["Padre / apoderado<br/><i>rol guardian</i>"]
    end

    subgraph PRESENTACION["🖥️ CAPA DE PRESENTACIÓN"]
        direction LR
        subgraph WEB["Aplicación web · Next.js"]
            direction TB
            W1["Panel del administrador general"]
            W2["Panel del administrador limitado"]
            W3["Portal familiar"]
        end
        subgraph APIREST["API REST · FastAPI"]
            direction TB
            SEG["AuthMiddleware<br/>JWT + sesión activa"]
            PER["require_permission<br/>RBAC por operación"]
            RUT["Rutas /api/*"]
            SEG --> PER --> RUT
        end
        WEB -->|"HTTP/JSON<br/>cookie httpOnly"| SEG
    end

    subgraph NEGOCIO["⚙️ CAPA DE LÓGICA DE NEGOCIO · módulos"]
        direction LR
        ACC["🔐 Acceso y seguridad<br/>auth · user · role · permission<br/>session · device · security_alert"]
        TER["🗺️ Territorio<br/>region · health_facility · user_region"]
        ATE["👶 Atención del niño<br/>child · guardian · child_guardian<br/>care_package · care_service · child_control"]
        REC["📝 Reclamos<br/>claim · claim_evidence"]
        POR["👪 Portal familiar<br/>portal"]
        INT["🔌 Integraciones<br/>identity (RENIEC)<br/>attention_sync (carga de atenciones)"]
        AUD["📜 Auditoría<br/>audit_log"]
    end

    subgraph PROCESOS["⏱️ PROCESOS PROGRAMADOS"]
        IMP["Importador de atenciones<br/><i>diario o a pedido</i>"]
    end

    subgraph DATOS["🗄️ CAPA DE DATOS"]
        direction LR
        BD[("PostgreSQL<br/>datos del negocio<br/>auditoría · triggers")]
        FS[("Archivos<br/>evidencias")]
    end

    subgraph EXTERNOS["🌐 SISTEMAS EXTERNOS"]
        direction LR
        RENIEC["API de RENIEC<br/><i>consulta de DNI</i>"]
        BDEXT[("BD externa de atenciones<br/><i>red de salud · solo lectura</i>")]
    end

    AG --> W1
    AL --> W2
    PA --> W3
    RUT --> NEGOCIO
    NEGOCIO --> BD
    REC --> FS
    INT -->|"HTTPS"| RENIEC
    IMP --> INT
    INT -->|"SQL de solo lectura"| BDEXT
    INT -->|"actualiza controles"| ATE
```

## Descripción

La arquitectura inicial se organiza en **tres capas principales**, más los procesos programados y los sistemas externos:

| Capa | Qué contiene | Responsabilidad |
|---|---|---|
| **Presentación** | Aplicación web (Next.js) con 3 interfaces, una por tipo de usuario, y la API REST (FastAPI) | Mostrar la información y recibir las peticiones. Antes de llegar al negocio, cada petición pasa por dos filtros: **autenticación** (JWT y sesión activa) y **autorización** (permiso de la operación). |
| **Lógica de negocio** | Los módulos del backend, agrupados en 7 áreas | Aplicar las reglas: quién puede hacer qué, cuándo un control es válido, qué ve cada padre, cómo se cargan las atenciones. |
| **Datos** | PostgreSQL y el almacenamiento de archivos | Guardar la información, reforzar la integridad y registrar la auditoría. |
| **Procesos programados** | Importador de atenciones | Traer cada cierto tiempo las atenciones de la base externa, sin bloquear a los usuarios. |
| **Sistemas externos** | API de RENIEC y BD externa de atenciones | Fuentes oficiales de identidad y de atenciones. |

## Áreas de la lógica de negocio

| Área | Módulos | Para qué sirve | Usuario principal |
|---|---|---|---|
| 🔐 Acceso y seguridad | `auth`, `user`, `role`, `permission`, `session`, `device`, `security_alert` | Quién entra, qué puede hacer y la detección de riesgos | Administrador general |
| 🗺️ Territorio | `region`, `health_facility`, `user_region` | Micro redes, establecimientos y asignación del personal | Administrador general |
| 👶 Atención del niño | `child`, `guardian`, `child_guardian`, `care_package`, `care_service`, `child_control` | El núcleo del negocio: niños, familias, catálogo y controles CRED | Administrador limitado |
| 📝 Reclamos | `claim`, `claim_evidence` | Reclamos de las familias y sus evidencias | Padre / Administrador limitado |
| 👪 Portal familiar | `portal` | Vista de los padres: sus hijos, sus controles y sus reclamos | Padre / apoderado |
| 🔌 Integraciones | `identity`, `attention_sync` | Consulta de DNI en RENIEC y carga de atenciones desde la BD externa | Administradores |
| 📜 Auditoría | `audit_log` | Consulta del historial de cambios | Administrador general |

## Flujos principales

| Flujo | Recorrido |
|---|---|
| Registrar un niño | Administrador limitado → panel → API → `identity` (consulta RENIEC) → `child` → PostgreSQL |
| Registrar un control | Administrador limitado → panel → API → `child_control` → PostgreSQL (trigger de auditoría) |
| Carga de atenciones | Importador → `attention_sync` → BD externa (lectura) → `child_control` → PostgreSQL |
| Consultar el portal | Padre → portal → API → `portal` → `child`, `child_control`, `claim` → PostgreSQL |
| Presentar un reclamo | Padre → portal → API → `portal` → `claim` / `claim_evidence` → PostgreSQL y archivos |

> Esta es la vista **inicial** (de alto nivel). Cómo se organiza cada módulo por dentro se explica en [Patrón arquitectónico](03-patron-arquitectonico.md) y en el [Modelo C4](../04-modelo-c4/nivel-3-componentes.md).
