# Componentes arquitectónicos

Componentes que forman SICNA, su responsabilidad y cómo se conectan.

## Diagrama de componentes

```mermaid
flowchart TB
    subgraph CLIENTE["🖥️ Navegador"]
        direction LR
        UI1["Panel administrador general"]
        UI2["Panel administrador limitado"]
        UI3["Portal familiar"]
    end

    subgraph NEXT["Servidor Next.js"]
        SA["Server actions<br/>reenvían la cookie de sesión"]
    end

    subgraph BACKEND["⚙️ API SICNA · FastAPI"]
        direction TB
        subgraph SEGURIDAD["Seguridad transversal"]
            direction LR
            MW["AuthMiddleware<br/>① JWT válido<br/>② sesión activa en BD<br/>③ contexto de auditoría"]
            PG["require_permission()<br/>④ permiso de la operación"]
            MW --> PG
        end

        subgraph MODS["Módulos de negocio · Clean Architecture"]
            direction LR
            M_ACC["Acceso y seguridad"]
            M_TER["Territorio"]
            M_ATE["Atención del niño"]
            M_REC["Reclamos"]
            M_POR["Portal familiar"]
            M_AUD["Auditoría"]
            M_INT["Integraciones<br/>identity · attention_sync"]
        end

        subgraph CORE["🔧 Núcleo compartido · core/"]
            direction LR
            C_DOM["core/domain<br/>errores comunes<br/>FileStoragePort"]
            C_DB["core/db<br/>conexión · translate_db_errors<br/>audit_context"]
            C_SEC["core/security<br/>JWT · Argon2 · cookies"]
            C_ST["core/storage<br/>LocalFileStorage"]
            C_HTTP["core/http<br/>respuestas y errores"]
        end

        PG --> MODS
        MODS --> CORE
    end

    subgraph WORKER["⏱️ Importador de atenciones"]
        JOB["Proceso programado<br/>usa AttentionSyncUseCase"]
    end

    subgraph DATOS["🗄️ Datos"]
        direction LR
        BD[("PostgreSQL<br/>tablas del negocio<br/>audit_log · triggers")]
        FS[("uploads/<br/>evidencias")]
    end

    subgraph EXT["🌐 Externos"]
        direction LR
        RENIEC["API RENIEC"]
        BDEXT[("BD externa de atenciones")]
    end

    UI1 & UI2 & UI3 --> SA
    SA -->|"HTTP/JSON · cookie httpOnly"| MW
    JOB --> M_INT
    C_DB --> BD
    C_ST --> FS
    M_INT -->|HTTPS| RENIEC
    M_INT -->|solo lectura| BDEXT
```

## Componentes

| Componente | Tipo | Responsabilidad | Ubicación |
|---|---|---|---|
| **Aplicación web** | Frontend | Tres interfaces: panel del administrador general, panel del administrador limitado y portal familiar | `srs-front/` |
| **Server actions** | Frontend (servidor) | Llamar a la API desde el servidor de Next.js, reenviando la cookie de sesión | `srs-front/lib/actions/` |
| **AuthMiddleware** | Seguridad | Leer el JWT, verificar que la sesión siga activa en la base de datos y preparar el contexto de auditoría | `core/security/auth_middleware.py` |
| **require_permission** | Seguridad | Verificar en cada ruta que el usuario tenga el permiso necesario; es lo que diferencia al administrador general del limitado | `core/security/permission_guard.py` |
| **Módulos de negocio** | Backend | Cada uno con sus 4 capas y su contenedor | `src/modules/` |
| **identity** *(diseñado)* | Integración | Consultar un DNI en RENIEC y devolver los datos de identidad | `src/modules/identity/` |
| **attention_sync** *(diseñado)* | Integración | Leer la base externa, aplicar las atenciones a los controles y registrar el resultado de cada carga | `src/modules/attention_sync/` |
| **Importador de atenciones** *(diseñado)* | Proceso programado | Ejecutar `attention_sync` cada noche o a pedido del administrador general | `srs-back/scripts/` |
| **Núcleo compartido** | Backend | Conexión a la base de datos, errores comunes, respuestas HTTP, JWT, contraseñas y archivos | `src/core/` |
| **PostgreSQL** | Datos | Tablas del negocio, auditoría y triggers de integridad | `src/core/db/sql/` |
| **Almacenamiento de evidencias** | Datos | Archivos JPG, PNG o PDF de los reclamos | carpeta `uploads/` |
| **API de RENIEC** | Externo | Fuente oficial de datos de identidad | Servicio de RENIEC |
| **BD externa de atenciones** | Externo | Atenciones registradas por los establecimientos | Red de salud |

## Piezas del núcleo compartido (`core/`)

| Pieza | Capa de Clean | Para qué sirve |
|---|---|---|
| `core/domain/persistence_errors.py` | Dominio (compartido) | Errores `AlreadyExistsError`, `ReferenceNotFoundError` e `InvalidDataError`, que usan todos los módulos |
| `core/domain/file_storage_port.py` | Dominio (compartido) | Puerto `FileStoragePort` para guardar y borrar archivos |
| `core/db/db_errors.py` | Infraestructura | Decorador `translate_db_errors` que convierte errores de asyncpg en errores del dominio |
| `core/db/audit_context.py` | Infraestructura | Envía usuario, dispositivo e IP a PostgreSQL para que la auditoría los registre |
| `core/storage/file_storage.py` | Infraestructura | `LocalFileStorage`, que implementa `FileStoragePort` sobre el disco |
| `core/security/` | Infraestructura | JWT, cookies, Argon2 (contraseñas) y tokens de dispositivo |

## Seguridad en capas (defensa en profundidad)

```mermaid
flowchart LR
    P["Petición"] --> L1["① Cookie httpOnly<br/>el JS no puede leer el token"]
    L1 --> L2["② JWT firmado<br/>y no expirado"]
    L2 --> L3["③ Sesión activa<br/>no revocada ni bloqueada"]
    L3 --> L4["④ Permiso de la operación<br/>RBAC"]
    L4 --> L5["⑤ Reglas del negocio<br/>ej.: el padre solo ve a sus hijos"]
    L5 --> L6["⑥ Restricciones y triggers<br/>en PostgreSQL"]
    L6 --> L7["⑦ Auditoría<br/>de todo cambio"]
```
