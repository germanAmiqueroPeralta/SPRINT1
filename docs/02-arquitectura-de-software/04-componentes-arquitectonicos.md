# Componentes arquitectónicos · Modelo C4

Vista de componentes de SICNA según el **Modelo C4 · Nivel 3 (componentes)**, con el estilo **monolito modular en capas**. Se abre el contenedor **API SICNA** y se muestran sus componentes: la API REST y los módulos del negocio, cómo se relacionan entre sí y con los contenedores y sistemas externos.

## Diagrama de componentes

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#ffffff", "primaryTextColor": "#111827", "primaryBorderColor": "#475569", "lineColor": "#64748b", "textColor": "#111827", "titleColor": "#111827", "edgeLabelBackground": "#ffffff", "clusterBkg": "#f8fafc", "clusterBorder": "#94a3b8"}, "flowchart": {"wrappingWidth": 600}}}%%
flowchart TB
    P1["<b>Administrador general</b><br/>[Persona]<br/>Gestiona usuarios, permisos,<br/>catálogo y auditoría"]
    P2["<b>Coordinador</b><br/>[Persona]<br/>Registra niños, controles<br/>y atiende reclamos"]
    P3["<b>Padre / apoderado</b><br/>[Persona]<br/>Consulta a sus hijos<br/>y presenta reclamos"]

    WEB["<b>Aplicación web</b><br/>[Contenedor: Next.js]<br/>Panel administrativo y portal familiar"]

    P1 & P2 & P3 -->|Usa| WEB

    subgraph API["API#nbsp;SICNA#nbsp;[Contenedor:#nbsp;Python#nbsp;+#nbsp;FastAPI#nbsp;·#nbsp;monolito#nbsp;modular]"]
        direction TB
        subgraph CPRES["CAPA#nbsp;DE#nbsp;PRESENTACIÓN"]
            REST["<b>API REST</b><br/>[Componente: rutas + middleware]<br/>Expone /api, autentica (JWT),<br/>verifica la sesión y el permiso"]
        end

        subgraph MODS["CAPA#nbsp;DE#nbsp;LÓGICA#nbsp;DE#nbsp;NEGOCIO#nbsp;—#nbsp;módulos"]
            direction TB
            ACC["<b>Acceso y seguridad</b><br/>[Componente: módulo]<br/>Usuarios, roles, permisos,<br/>sesiones y alertas"]
            TER["<b>Territorio</b><br/>[Componente: módulo]<br/>Micro redes y establecimientos"]
            NIN["<b>Niños y familias</b><br/>[Componente: módulo]<br/>Padrón de niños y sus padres"]
            CAT["<b>Catálogo CRED</b><br/>[Componente: módulo]<br/>Paquetes y prestaciones por edad"]
            CON["<b>Controles CRED</b><br/>[Componente: módulo]<br/>Controles programados,<br/>realizados y vencidos"]
            REC["<b>Reclamos</b><br/>[Componente: módulo]<br/>Reclamos y sus evidencias"]
            POR["<b>Portal familiar</b><br/>[Componente: módulo]<br/>Vista del padre sobre sus hijos"]
            AUD["<b>Auditoría</b><br/>[Componente: módulo]<br/>Historial de cambios"]
            INT["<b>Integraciones</b><br/>[Componente: módulo]<br/>Consulta de DNI y<br/>carga de atenciones"]
        end
    end

    DB[("<b>Base de datos</b><br/>[Contenedor: PostgreSQL]<br/>Datos del negocio y auditoría")]
    FS["<b>Almacenamiento de archivos</b><br/>[Contenedor: disco del servidor]<br/>Evidencias de reclamos"]
    RENIEC["<b>API de RENIEC</b><br/>[Sistema externo]<br/>Datos de identidad por DNI"]
    BDEXT["<b>BD externa de atenciones</b><br/>[Sistema externo]<br/>Atenciones de los establecimientos"]

    WEB -->|"Llama a la API<br/>[HTTPS / JSON]"| REST
    REST -->|Valida sesión y permiso| ACC
    REST -->|Invoca casos de uso| POR & AUD & INT

    NIN -->|Valida establecimiento| TER
    NIN -->|Autocompleta por DNI| INT
    CON -->|Consulta niño| NIN
    CON -->|Consulta prestación| CAT
    REC -->|Verifica niño| NIN
    POR -->|Consulta avance| CON
    POR -->|Presenta reclamo| REC
    INT -->|Actualiza controles| CON

    INT -->|"Consulta DNI<br/>[HTTPS/REST]"| RENIEC
    INT -->|"Lee atenciones<br/>[SQL · solo lectura]"| BDEXT
    REC -->|Guarda evidencias| FS
    API -->|"Lee y escribe<br/>[SQL]"| DB

    classDef person fill:#08427b,stroke:#052e56,color:#ffffff
    classDef container fill:#1168bd,stroke:#0b4884,color:#ffffff
    classDef component fill:#85bbf0,stroke:#5d82a8,color:#000000
    classDef external fill:#999999,stroke:#6b6b6b,color:#ffffff
    class P1,P2,P3 person
    class WEB,DB,FS container
    class REST,ACC,TER,NIN,CAT,CON,REC,POR,AUD,INT component
    class RENIEC,BDEXT external
    style API fill:#f8fafc,stroke:#334155,stroke-width:2px,stroke-dasharray:8 5,color:#0f172a
    style CPRES fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a8a
    style MODS fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
```

### Leyenda

| Color | Elemento |
|---|---|
| Azul oscuro | Persona |
| Azul | Contenedor |
| Celeste | Componente |
| Gris | Sistema externo |

Flecha: relación con verbo y [protocolo]. Línea discontinua: límite del contenedor.

### Notas

- Cada componente es un módulo del monolito: todos se despliegan juntos.
- Un módulo usa a otro solo a través de su interfaz pública (su servicio), nunca sus tablas.
- El **Importador de atenciones** (proceso programado, contenedor aparte) no aparece en el diagrama: ejecuta cada noche el mismo módulo de Integraciones, sin pasar por la API REST, para no bloquear a los usuarios.

## Componentes

| Componente | Tipo | Responsabilidad |
|---|---|---|
| **API REST** | Componente | Recibir las peticiones, autenticar al usuario, verificar su sesión y su permiso, y derivar al módulo correspondiente |
| **Acceso y seguridad** | Módulo | Usuarios, roles, permisos, sesiones, dispositivos y alertas de seguridad |
| **Territorio** | Módulo | Micro redes, establecimientos de salud y asignación del personal |
| **Niños y familias** | Módulo | Padrón de niños, sus padres o apoderados y la relación entre ellos |
| **Catálogo CRED** | Módulo | Paquetes y prestaciones de atención según la edad del niño |
| **Controles CRED** | Módulo | Controles de cada niño y su estado (programado, realizado, vencido) |
| **Reclamos** | Módulo | Reclamos de las familias y sus archivos de evidencia |
| **Portal familiar** | Módulo | Vista del padre: solo sus hijos, sus controles y sus reclamos |
| **Auditoría** | Módulo | Consulta del historial de cambios (quién, qué y cuándo) |
| **Integraciones** | Módulo | Consulta de DNI en RENIEC y carga de atenciones desde la BD externa |
| **Importador de atenciones** | Contenedor (proceso programado) | Ejecutar la carga de atenciones cada noche o a pedido del administrador general |
| **Base de datos** | Contenedor | Datos del negocio, auditoría y reglas de integridad |
| **Almacenamiento de archivos** | Contenedor | Archivos JPG, PNG o PDF de los reclamos |
| **API de RENIEC** | Sistema externo | Fuente oficial de datos de identidad |
| **BD externa de atenciones** | Sistema externo | Atenciones registradas por los establecimientos de la red de salud |

## Seguridad en capas (defensa en profundidad)

```mermaid
flowchart LR
    P["Petición"] --> L1["① Cookie httpOnly<br/>el JS no puede leer el token"]
    L1 --> L2["② JWT firmado<br/>y no expirado"]
    L2 --> L3["③ Sesión activa<br/>no revocada ni bloqueada"]
    L3 --> L4["④ Permiso de la operación<br/>RBAC"]
    L4 --> L5["⑤ Reglas del negocio<br/>ej.: el padre solo ve a sus hijos"]
    L5 --> L6["⑥ Restricciones<br/>en la base de datos"]
    L6 --> L7["⑦ Auditoría<br/>de todo cambio"]
```
