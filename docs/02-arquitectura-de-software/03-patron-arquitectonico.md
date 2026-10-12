# Patrón arquitectónico

Se utiliza el patrón arquitectónico **Clean Architecture** (Arquitectura Limpia).

| Elemento | Descripción aplicada a SICNA |
|---|---|
| Patrón / enfoque arquitectónico | Clean Architecture, aplicada **dentro de cada módulo** del monolito. |
| Objetivo | Separar responsabilidades y hacer que las reglas del negocio (por ejemplo, "un control realizado debe tener fecha") no dependan de FastAPI, de PostgreSQL, de RENIEC ni de la base externa de atenciones. |
| ¿Qué problema resuelve? | Evita el acoplamiento entre la interfaz web, las reglas del negocio y las tecnologías externas (base de datos, API de RENIEC, BD externa de atenciones y almacenamiento de archivos). Sin esta separación, un cambio en el proveedor o en el formato de un sistema externo obligaría a modificar el negocio. |
| Capas definidas | Presentación, Aplicación, Dominio e Infraestructura. |
| Regla de dependencia | Las dependencias del código apuntan siempre **hacia el dominio**. El dominio no conoce a ninguna otra capa. |
| Beneficios | • Facilita el mantenimiento y las pruebas unitarias: las reglas se prueban sin base de datos ni RENIEC.<br>• Cambiar una tecnología o un proveedor externo solo requiere un adaptador nuevo.<br>• Todos los módulos siguen la misma estructura. |

## Diagrama de Clean Architecture en SICNA

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 600}}}%%
flowchart TB
    U(["👤 Usuario<br/>(navegador)"]) --> FE["«frontend»<br/><b>Aplicación web</b><br/>panel administrativo · portal familiar"]

    subgraph SICNA["«aplicación»#nbsp;SICNA#nbsp;Backend#nbsp;[Python#nbsp;·#nbsp;FastAPI]"]
        direction TB
        subgraph PRES["CAPA#nbsp;DE#nbsp;PRESENTACIÓN#nbsp;—#nbsp;depende#nbsp;de#nbsp;FastAPI"]
            direction LR
            p1["«middleware»<br/><b>Seguridad</b><br/>autenticación · sesión · permisos"]
            p2["«ruta»<br/><b>Rutas REST</b><br/>endpoints por módulo"]
            p3["«controlador»<br/><b>Controladores</b><br/>validan la entrada e invocan el caso de uso"]
            p4["«presentador»<br/><b>Presentadores</b><br/>entidad → JSON · error → código HTTP"]
            p1 ~~~ p2 ~~~ p3 ~~~ p4
        end

        subgraph UC["CAPA#nbsp;DE#nbsp;APLICACIÓN#nbsp;—#nbsp;casos#nbsp;de#nbsp;uso"]
            direction LR
            a1["«caso de uso»<br/><b>RegistrarNiñoCasoUso</b><br/>(ejecuta)"]
            a2["«caso de uso»<br/><b>RegistrarControlCasoUso</b><br/>(ejecuta)"]
            a3["«caso de uso»<br/><b>PresentarReclamoCasoUso</b><br/>(ejecuta)"]
            a4["«caso de uso»<br/><b>CargarAtencionesCasoUso</b><br/>(ejecuta)"]
            a1 ~~~ a2 ~~~ a3 ~~~ a4
        end

        subgraph DOM["CAPA#nbsp;DE#nbsp;DOMINIO#nbsp;—#nbsp;núcleo"]
            direction TB
            subgraph MOD["Modelos#nbsp;(entidades#nbsp;y#nbsp;reglas)"]
                direction LR
                d1["«entidad»<br/><b>Niño</b><br/>datos, fecha de nacimiento, condición al nacer"]
                d2["«entidad»<br/><b>ControlCRED</b><br/>prestación · estado · fechas"]
                d3["«entidad»<br/><b>Reclamo</b><br/>motivo · estado · evidencias"]
                d4["«reglas»<br/><b>Reglas CRED</b><br/>edad del niño · estado del control"]
                d1 ~~~ d2 ~~~ d3 ~~~ d4
            end
            subgraph CON["Contratos#nbsp;(puertos)"]
                direction LR
                c1["«interfaz»<br/><b>RepositorioNiños</b>"]
                c2["«interfaz»<br/><b>RepositorioControles</b>"]
                c3["«interfaz»<br/><b>RepositorioReclamos</b>"]
                c4["«interfaz»<br/><b>ProveedorIdentidad</b>"]
                c5["«interfaz»<br/><b>FuenteAtenciones</b>"]
                c6["«interfaz»<br/><b>AlmacenArchivos</b>"]
                c1 ~~~ c2 ~~~ c3 ~~~ c4 ~~~ c5 ~~~ c6
            end
            MOD ~~~ CON
        end

        subgraph INFRA["CAPA#nbsp;DE#nbsp;INFRAESTRUCTURA#nbsp;—#nbsp;depende#nbsp;de#nbsp;PostgreSQL#nbsp;y#nbsp;servicios#nbsp;externos"]
            direction LR
            i1["«adaptador»<br/><b>RepositorioNiñosPostgres</b>"]
            i2["«adaptador»<br/><b>RepositorioControlesPostgres</b>"]
            i3["«adaptador»<br/><b>RepositorioReclamosPostgres</b>"]
            i4["«adaptador»<br/><b>ReniecApiAdaptador</b>"]
            i5["«adaptador»<br/><b>BDExternaAtencionesAdaptador</b>"]
            i6["«adaptador»<br/><b>AlmacenArchivosLocal</b>"]
            i1 ~~~ i2 ~~~ i3 ~~~ i4 ~~~ i5 ~~~ i6
        end

        CR["«raíz de composición»<br/><b>Contenedor de dependencias</b><br/>único punto que elige qué adaptador cumple cada contrato<br/>y lo inyecta en los casos de uso"]
    end

    DB[("«base de datos»<br/><b>PostgreSQL</b>")]
    EXT["«sistemas externos»<br/><b>API de RENIEC</b><br/><b>BD externa de atenciones</b>"]
    FS[("«almacenamiento»<br/><b>Archivos de evidencias</b>")]

    FE -->|HTTP/JSON| PRES
    PRES -->|invoca| UC
    UC -.->|usa| MOD
    CON -.->|"son implementados por"| INFRA
    CR ~~~ PRES
    INFRA -->|SQL| DB
    INFRA -->|HTTPS · SQL| EXT
    INFRA -->|archivos| FS

    classDef card fill:#ffffff,stroke:#444,color:#111
    classDef ext fill:#e6e6e6,stroke:#444,color:#111
    class p1,p2,p3,p4,a1,a2,a3,a4,d1,d2,d3,d4,c1,c2,c3,c4,c5,c6,i1,i2,i3,i4,i5,i6,CR card
    class U,FE,DB,EXT,FS ext

    style SICNA fill:#ffffff,stroke:#222,stroke-dasharray:8 5
    style PRES fill:#dbeafe,stroke:#3b6fd8
    style UC fill:#e3f5e1,stroke:#2e8b3d
    style DOM fill:#fdf1c7,stroke:#c9a227
    style MOD fill:none,stroke:#c9a227,stroke-dasharray:3 3
    style CON fill:none,stroke:#c9a227,stroke-dasharray:3 3
    style INFRA fill:#ece3f7,stroke:#7b4bb7

    linkStyle 23 stroke:#2e8b3d,stroke-width:2px
    linkStyle 24 stroke:#7b4bb7,stroke-width:2px
```

### Reglas de dependencia

1. El dominio no importa nada de las capas externas.
2. Los casos de uso solo conocen entidades y contratos (puertos).
3. Los adaptadores implementan los contratos y son intercambiables.
4. Cambiar de tecnología o de proveedor significa cambiar el contenedor de dependencias, no el dominio.
