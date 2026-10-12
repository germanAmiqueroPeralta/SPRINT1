# Nivel 3 · Diagrama de componentes

> **Proyecto:** SICNA · **Contenedor detallado:** API SICNA (y el importador, que usa los mismos módulos) · **Modelo:** C4 · Nivel 3 de 4

---

## 1. Objetivo

Abrir el contenedor **API SICNA** del nivel 2 y mostrar sus **componentes**: los módulos del monolito, qué hace cada uno y cómo se relacionan.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Cómo se divide el backend por dentro y qué hace cada parte? |
| Audiencia | Arquitectos y desarrolladores |
| Qué es un componente | En SICNA, cada **módulo** del monolito es un componente |
| Nivel anterior | [Nivel 2 · Contenedores](nivel-2-contenedores.md) |
| Siguiente nivel | [Nivel 4 · Código](nivel-4-codigo.md) |

---

## 2. Diagrama general

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#ffffff", "primaryTextColor": "#111827", "primaryBorderColor": "#475569", "lineColor": "#64748b", "textColor": "#111827", "titleColor": "#111827", "edgeLabelBackground": "#ffffff", "clusterBkg": "#f8fafc", "clusterBorder": "#94a3b8"}, "flowchart": {"wrappingWidth": 600}}}%%
flowchart TB
    Web["<b>Aplicación web</b>"]
    Job["<b>Importador de atenciones</b><br/>proceso programado"]

    subgraph API["API#nbsp;SICNA#nbsp;·#nbsp;monolito#nbsp;modular"]
        direction TB
        Entrada["<b>Entrada HTTP</b><br/>AuthMiddleware · require_permission<br/>routes.py (/api)"]

            subgraph Acceso["🔐#nbsp;Acceso#nbsp;y#nbsp;seguridad"]
                direction LR
                auth[auth] ~~~ user[user] ~~~ role[role] ~~~ perm[permission] ~~~ session[session] ~~~ device[device] ~~~ alert[security_alert]
            end
            subgraph Territorio["🗺️#nbsp;Territorio"]
                direction LR
                region[region] ~~~ hf[health_facility] ~~~ ur[user_region]
            end
            subgraph Atencion["👶#nbsp;Atención#nbsp;del#nbsp;niño"]
                direction LR
                child[child] ~~~ guardian[guardian] ~~~ cg[child_guardian] ~~~ pkg[care_package] ~~~ srv[care_service] ~~~ ctrl[child_control]
            end
            subgraph Reclamos["📝#nbsp;Reclamos"]
                direction LR
                claim[claim] ~~~ ev[claim_evidence]
            end
            subgraph Transv["👪#nbsp;Portal#nbsp;y#nbsp;📜#nbsp;auditoría"]
                direction LR
                portal[portal] ~~~ audit[audit_log]
            end
            subgraph Integraciones["🔌#nbsp;Integraciones#nbsp;(diseñado)"]
                direction LR
                identity[identity] ~~~ sync[attention_sync]
            end
            Entrada -->|enruta cada petición al módulo| Acceso
            Acceso ~~~ Territorio ~~~ Atencion ~~~ Reclamos ~~~ Transv ~~~ Integraciones
    end

    BD[("<b>PostgreSQL</b>")]
    FS[("<b>Archivos</b><br/>evidencias")]
    RENIEC["<b>API RENIEC</b>"]
    BDEXT[("<b>BD externa</b><br/>de atenciones")]

    Web -->|HTTP/JSON| Entrada
    Integraciones ~~~ Job
    Job -.->|ejecuta attention_sync| Integraciones
    Integraciones ~~~ BD
    API -->|SQL| BD
    Reclamos -->|guarda| FS
    Integraciones -->|"identity · HTTPS"| RENIEC
    Integraciones -->|"attention_sync · solo lectura"| BDEXT

    classDef mod fill:#ffffff,stroke:#475569,color:#111827
    classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
    classDef entry fill:#1e3a8a,stroke:#1e3a8a,color:#ffffff
    class auth,user,role,perm,session,device,alert,region,hf,ur,child,guardian,cg,pkg,srv,ctrl,claim,ev,portal,audit,identity,sync mod
    class Web,Job,BD,FS,RENIEC,BDEXT ext
    class Entrada entry
    style API fill:#f8fafc,stroke:#334155,stroke-width:2px,stroke-dasharray:8 5,color:#0f172a
    style Acceso fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a8a
    style Territorio fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#4c1d95
    style Atencion fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style Reclamos fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    style Transv fill:#e0f2fe,stroke:#0369a1,stroke-width:2px,color:#0c4a6e
    style Integraciones fill:#fce7f3,stroke:#db2777,stroke-width:2px,stroke-dasharray:6 4,color:#831843
```

> El diagrama general muestra **cómo se agrupan** los módulos. Las dependencias entre ellos se muestran por separado abajo para que se lean con claridad.

---

## 3. Dependencias entre módulos

Cada flecha significa "**usa a**", siempre mediante un **puerto** declarado en el dominio del módulo que llama y un **adaptador** en su infraestructura.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#ffffff", "primaryTextColor": "#111827", "primaryBorderColor": "#475569", "lineColor": "#64748b", "textColor": "#111827", "titleColor": "#111827", "edgeLabelBackground": "#ffffff", "clusterBkg": "#f8fafc", "clusterBorder": "#94a3b8"}, "flowchart": {"wrappingWidth": 600}}}%%
flowchart TB
    portal[portal]
    claim[claim]
    ev[claim_evidence]
    sync[attention_sync]
    identity[identity]
    cg[child_guardian]
    ctrl[child_control]
    child[child]
    guardian[guardian]
    srv[care_service]
    pkg[care_package]
    hf[health_facility]
    user[user]
    region[region]
    role[role]

    portal --> claim & ev & ctrl & pkg & srv & hf
    ev --> claim
    claim --> child & srv & hf
    sync -.->|diseñado| child & srv & ctrl
    cg --> child & guardian
    ctrl --> child & srv & pkg & hf
    child -.->|diseñado| identity
    guardian -.->|diseñado| identity
    child --> hf
    guardian --> user & role
    srv --> pkg
    hf --> region
    user --> role

    classDef rec fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    classDef int fill:#fce7f3,stroke:#db2777,stroke-width:2px,stroke-dasharray:6 4,color:#831843
    classDef ate fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    classDef bas fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#4c1d95
    class portal,claim,ev rec
    class sync,identity int
    class cg,ctrl,child,guardian,srv,pkg ate
    class hf,user,region,role bas
```

**Colores:** 🟨 Reclamos y portal · 🟩 Atención del niño · 🟪 Territorio y acceso · 🩷 Integraciones (borde punteado: diseñado, aún no programado).

| Origen | Destino | Para qué | Puerto que usa el origen |
|---|---|---|---|
| child | health_facility | Validar que el establecimiento exista y esté activo | `HealthFacilityPort` |
| child, guardian | identity *(diseñado)* | Autocompletar y validar datos con RENIEC | `IdentityLookupPort` |
| child_control | child, care_service, care_package, health_facility | Validar las reglas del control | `ChildPort`, `CareServicePort`, `CarePackagePort`, `HealthFacilityPort` |
| attention_sync *(diseñado)* | child, care_service, child_control | Emparejar cada atención y actualizar el control | `ChildLookupPort`, `CareServiceLookupPort`, `ChildControlPort` |
| claim | child, care_service, health_facility | Validar el reclamo y completar datos del niño | `ChildPort`, `CareServicePort`, `HealthFacilityPort` |
| claim_evidence | claim | Verificar que el reclamo exista y no esté cerrado | `ClaimPort` |
| guardian | user, role | Crear la cuenta del portal del padre | `UserAccountPort`, `PasswordHasherPort` |
| portal | claim, claim_evidence, child_control, catálogo | Armar la vista del padre | `ClaimsPort`, `EvidencesPort`, `CatalogPort` |
| auth | user, role, device, session, security_alert | Iniciar sesión de forma segura | `UserPort`, `DevicePort`, `SessionPort`, `SecurityScanPort`… |

---

## 4. Componentes

| Componente | Responsabilidad | Ruta de la API | Usuario principal |
|---|---|---|---|
| **Entrada HTTP** | Autenticación, permisos, registro de rutas y manejo de errores | `/api` | Todos |
| **auth** | Iniciar sesión, cerrar sesión, usuario actual | `/api/auth` | Todos |
| **user** · **role** · **permission** | Cuentas, roles y permisos (RBAC) | `/api/users`, `/api/roles`, `/api/permissions` | Administrador general |
| **session** · **device** · **security_alert** | Sesiones, dispositivos y alertas de riesgo | `/api/sessions`, `/api/devices`, `/api/security-alerts` | Administrador general |
| **region** · **health_facility** · **user_region** | Micro redes, establecimientos y su asignación al personal | `/api/regions`, `/api/health-facilities`, `/api/user-regions` | Administrador general |
| **child** | Niños del padrón | `/api/children` | Coordinador |
| **guardian** · **child_guardian** | Padres o apoderados y su vínculo con los niños | `/api/guardians`, `/api/child-guardians` | Coordinador |
| **care_package** · **care_service** | Catálogo SISNAT: paquetes y prestaciones por edad | `/api/care-packages`, `/api/care-services` | Administrador general |
| **child_control** | Controles CRED de cada niño | `/api/child-controls` | Coordinador |
| **claim** · **claim_evidence** | Reclamos y sus evidencias | `/api/claims`, `/api/claim-evidences` | Coordinador |
| **portal** | Vista del padre: sus hijos, controles y reclamos | `/api/portal` | Padre / apoderado |
| **audit_log** | Consulta de la auditoría | `/api/audit-logs` | Administrador general |
| **identity** *(diseñado)* | Consulta de DNI en RENIEC | `/api/identity/{dni}` | Administradores |
| **attention_sync** *(diseñado)* | Carga de atenciones e historial de cargas | `/api/attention-sync` | Administrador general |

---

## 5. Relaciones con datos y sistemas externos

| Componente | Destino | Descripción | Patrón aplicado |
|---|---|---|---|
| Todos los módulos | PostgreSQL | Cada módulo lee y escribe sus propias tablas | Repository |
| claim, claim_evidence | Archivos | Guardar y borrar evidencias | Adapter (`FileStoragePort` → `LocalFileStorage`) |
| identity | API de RENIEC | Consultar datos por DNI | Adapter (`IdentityProviderPort` → `ReniecApiAdapter`) + caché |
| attention_sync | BD externa de atenciones | Leer atenciones nuevas | Adapter (`AttentionSourcePort` → `ExternalDbAttentionSource`) |
| Entrada HTTP | PostgreSQL | Enviar usuario, dispositivo e IP para la auditoría | Contexto de auditoría |

---

## 6. Reglas del monolito modular

| # | Regla | Motivo |
|---|---|---|
| 1 | Todos los módulos se despliegan juntos; el importador reutiliza el mismo código | Es un monolito: una sola base de código |
| 2 | Un módulo usa a otro solo mediante un **puerto propio** y un **adaptador** | Un cambio interno en un módulo no rompe a los demás |
| 3 | El dominio y la aplicación de un módulo nunca importan las capas `application/`, `infrastructure/` ni `presentation/` de otro | Se cumple la regla de dependencia de Clean |
| 4 | Cada sistema externo se integra en **un solo módulo** (RENIEC en `identity`, la BD externa en `attention_sync`) | La integración queda en un solo lugar y es fácil de reemplazar |
| 5 | Cada módulo se organiza por dentro con Clean Architecture | Las reglas del negocio no dependen de la tecnología (ver nivel 4) |

> **Excepción conocida:** `portal` es un módulo **agregador** (arma la vista del padre a partir de varios módulos), así que sus puertos devuelven directamente las entidades de dominio de los otros módulos. Depender del dominio de otro módulo es aceptable, porque es la capa más estable.
