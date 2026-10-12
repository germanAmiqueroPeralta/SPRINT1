# Patrón arquitectónico

Se utiliza el patrón arquitectónico **Clean Architecture** (Arquitectura Limpia).

| Elemento | Descripción aplicada a SICNA |
|---|---|
| Patrón / enfoque arquitectónico | Clean Architecture, aplicada **dentro de cada módulo** del monolito. |
| Objetivo | Que las reglas del negocio (por ejemplo, "un control realizado debe tener fecha") no dependan de FastAPI, de PostgreSQL, de RENIEC ni de la base externa de atenciones. |
| ¿Qué problema resuelve? | Antes, la lógica estaba mezclada: los handlers capturaban errores de PostgreSQL, los servicios importaban los repositorios concretos y las entidades sabían leer filas de la base de datos. Además, con RENIEC y la base externa, sus formatos podrían contaminar el negocio. |
| Capas definidas | Dominio, Aplicación, Infraestructura y Presentación. |
| Regla de dependencia | El código solo importa **hacia el dominio**. El dominio no conoce a nadie. |
| Beneficios | • Las reglas se pueden probar sin base de datos ni RENIEC.<br>• Cambiar una tecnología o un proveedor externo solo requiere un adaptador nuevo.<br>• Todos los módulos tienen la misma estructura. |

## Los círculos de Clean Architecture en SICNA

```mermaid
flowchart TB
    subgraph F["🔵 FRAMEWORKS Y DRIVERS · lo más externo y cambiante"]
        direction TB
        subgraph FR[" "]
            direction LR
            f1["FastAPI · Uvicorn"] ~~~ f2["asyncpg · PostgreSQL"] ~~~ f3["Cliente HTTP → API RENIEC"] ~~~ f4["Driver de la BD externa"] ~~~ f5["Disco (uploads)"]
        end

        subgraph IA["🟢 ADAPTADORES DE INTERFAZ · traducen entre el mundo externo y el negocio"]
            direction TB
            subgraph IAR[" "]
                direction LR
                subgraph PRES["presentation/"]
                    direction LR
                    ia1["Rutas"] ~~~ ia2["Handlers"] ~~~ ia3["Presenters<br/>entidad → JSON"]
                end
                subgraph INFRA["infrastructure/"]
                    direction LR
                    ia4["Repositorios SQL"] ~~~ ia5["Mappers<br/>fila → entidad"] ~~~ ia6["ReniecApiAdapter"] ~~~ ia7["ExternalDbAttentionSource"]
                end
            end

            subgraph APP["🟡 APLICACIÓN · casos de uso"]
                direction TB
                subgraph AR[" "]
                    direction LR
                    a1["ChildControlCreateUseCase"] ~~~ a2["IdentityLookupUseCase"] ~~~ a3["AttentionSyncUseCase"]
                end

                subgraph DOM["🔴 DOMINIO · reglas y contratos"]
                    direction LR
                    d1["Entidades<br/>Child · ChildControl · Claim"] ~~~ d2["Puertos (Protocol)<br/>ChildControlRepositoryPort<br/>IdentityProviderPort<br/>AttentionSourcePort"] ~~~ d3["Errores<br/>ChildControlValidationError"]
                end
            end
        end
    end

    FR ~~~ IA
    IAR ~~~ APP
    AR ~~~ DOM

    style F fill:#dbeafe,stroke:#1d4ed8
    style IA fill:#dcfce7,stroke:#15803d
    style APP fill:#fef9c3,stroke:#a16207
    style DOM fill:#fee2e2,stroke:#b91c1c
    style FR fill:none,stroke:none
    style IAR fill:none,stroke:none
    style AR fill:none,stroke:none
```

> Cada círculo solo conoce a los que están **dentro** de él. El dominio (rojo) no sabe que existen FastAPI, PostgreSQL ni RENIEC.

| Círculo | Carpeta en SICNA | Qué contiene |
|---|---|---|
| 🔴 **Dominio** | `domain/` | Entidades (`ChildControl`), errores, puertos (`ChildControlRepositoryPort`, `ChildPort`, `IdentityProviderPort`, `AttentionSourcePort`) y referencias (`ChildRef`) |
| 🟡 **Aplicación** | `application/` | Casos de uso (`ChildControlCreateUseCase`, `AttentionSyncUseCase`…) y validaciones |
| 🟢 **Adaptadores de interfaz** | `presentation/` y `infrastructure/` | Rutas, handlers y presenters; repositorios, mappers y adaptadores (`ReniecApiAdapter`, `ExternalDbAttentionSource`) |
| 🔵 **Frameworks y drivers** | `main.py`, `core/`, `*_container.py` | FastAPI, asyncpg, cliente HTTP, configuración y conexión de las piezas |

## Regla de dependencia: cómo viaja una petición

Las **llamadas** van de afuera hacia adentro y de vuelta, pero las **dependencias del código** siempre apuntan hacia el dominio:

```mermaid
flowchart LR
    U["Usuario"] -->|1 · HTTP| R["Ruta + Handler<br/><i>presentation</i>"]
    R -->|2 · ejecuta| UC["Caso de uso<br/><i>application</i>"]
    UC -->|3 · usa la interfaz| PORT[["Puerto<br/><i>domain</i>"]]
    ADP["Adaptador / Repositorio<br/><i>infrastructure</i>"] -.->|implementa| PORT
    ADP -->|4 · SQL / HTTPS| EXT[("PostgreSQL · RENIEC<br/>BD externa")]
    CONT["Contenedor<br/><i>composición</i>"] -.->|inyecta el adaptador| UC
```

## Antes y después

| Aspecto | Antes (capas simples) | Después (Clean Architecture) |
|---|---|---|
| Servicio → repositorio | El servicio importaba la clase concreta con SQL | El caso de uso recibe una **interfaz** (`Protocol`) por el constructor |
| Errores de base de datos | Los handlers capturaban `UniqueViolationError` de asyncpg | El repositorio los traduce a errores propios (`AlreadyExistsError`) |
| Entidad | Tenía `from_row()` (leía filas) y `to_dict()` (armaba el JSON) | Es un `dataclass` puro; el mapper y el presenter están en sus capas |
| Módulo → otro módulo | Importaba el servicio del otro módulo directamente | Declara un **puerto** con lo mínimo que necesita; un adaptador lo implementa |
| Sistemas externos | — | RENIEC y la BD externa quedan detrás de `IdentityProviderPort` y `AttentionSourcePort` |

> **Cómo comprobarlo:** en ningún archivo de `domain/` ni de `application/` aparece `import fastapi`, `asyncpg` ni nada de `infrastructure/`. Lo verificamos con una búsqueda automática (ver [Verificación](../05-verificacion/01-verificacion-de-la-arquitectura.md)).
