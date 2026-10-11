# Nivel 4 · Diagrama de código

> **Proyecto:** SICNA · **Módulo detallado:** Controles CRED (`child_control`) · **Modelo:** C4 · Nivel 4 de 4 · **Notación:** UML

---

## 1. Objetivo

Mostrar **cómo está organizado el código** de un componente del nivel 3: sus clases, interfaces y cómo colaboran. Se detalla el flujo **Registrar control CRED**.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué clases existen dentro de un módulo y cómo se relacionan? |
| Audiencia | Desarrolladores |
| Por qué solo un módulo | El nivel 4 es opcional en C4 y se dibuja solo para los componentes más importantes. Los otros 18 módulos siguen la misma estructura |
| Nivel anterior | [Nivel 3 · Componentes](nivel-3-componentes.md) |

---

## 2. Diagrama de clases

```mermaid
classDiagram
    direction LR

    %% Presentación
    class create_child_control_handler {
        <<handler>>
        +create(service, request, body)
    }
    class ChildControlService {
        <<facade>>
        +create(control)
        +get_all(child_id, service_id, status)
        +get_by_id(id)
        +update(control, current)
        +delete(id)
    }

    %% Aplicación
    class ChildControlCreateUseCase {
        -repository: ChildControlRepositoryPort
        -refs: ChildControlRefs
        +execute(control) ChildControl
    }
    class ChildControlRefs {
        +children: ChildPort
        +care_services: CareServicePort
        +care_packages: CarePackagePort
        +health_facilities: HealthFacilityPort
    }

    %% Dominio
    class ChildControl {
        <<entity>>
        +id: UUID
        +child_id: UUID
        +service_id: UUID
        +status: ControlStatus
        +scheduled_date: date
        +completed_date: date
    }
    class ChildControlRepositoryPort {
        <<interface>>
        +create(control) ChildControl
        +get_by_id(id) ChildControl
        +update(control) ChildControl
        +delete(id) str
    }
    class ChildPort {
        <<interface>>
        +get_by_id(child_id) ChildRef
    }
    class CareServicePort {
        <<interface>>
        +get_by_id(id) CareServiceRef
    }
    class ChildRef {
        <<value object>>
        +active: bool
        +birth_date: date
        +birth_condition: str
    }

    %% Infraestructura
    class AsyncpgChildControlRepository {
        +create(control) ChildControl
        +get_by_id(id) ChildControl
    }
    class ChildAdapter {
        +get_by_id(child_id) ChildRef
    }
    class CareServiceAdapter {
        +get_by_id(id) CareServiceRef
    }

    create_child_control_handler ..> ChildControlService : usa
    ChildControlService *-- ChildControlCreateUseCase
    ChildControlCreateUseCase --> ChildControlRepositoryPort
    ChildControlCreateUseCase --> ChildControlRefs
    ChildControlCreateUseCase ..> ChildControl : valida y guarda
    ChildControlRefs --> ChildPort
    ChildControlRefs --> CareServicePort
    ChildPort ..> ChildRef
    AsyncpgChildControlRepository ..|> ChildControlRepositoryPort
    ChildAdapter ..|> ChildPort
    CareServiceAdapter ..|> CareServicePort
```

### 2.1 Capas de Clean Architecture

| Capa | Contiene | Regla |
|---|---|---|
| **Presentación** | Handlers, rutas, presenter | Traduce HTTP a llamadas del caso de uso y viceversa |
| **Aplicación** | Casos de uso, `ChildControlRefs`, validaciones | Coordina los pasos; no conoce la tecnología |
| **Dominio** | `ChildControl`, puertos, `ChildRef` | Reglas y contratos; no importa nada de las otras capas |
| **Infraestructura** | Repositorio asyncpg, adaptadores | Implementa los puertos del dominio |

> **Regla de dependencia:** las flechas apuntan hacia el dominio. La infraestructura implementa los puertos (línea discontinua con triángulo); el dominio nunca conoce a la infraestructura.

### 2.2 Clases del módulo

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `create_child_control_handler` | Handler | Presentación | Arma la entidad a partir del JSON y responde 201, 400 o 409 |
| `ChildControlService` | Fachada | Aplicación | Agrupa los 5 casos de uso del módulo |
| `ChildControlCreateUseCase` | Caso de uso | Aplicación | Valida las reglas RN-01 a RN-07 y guarda el control |
| `ChildControlRefs` | Parameter Object | Aplicación | Agrupa los 4 puertos hacia otros módulos |
| `ChildControl` | Entidad | Dominio | Representa un control CRED |
| `ChildControlRepositoryPort` | Interfaz | Dominio | Contrato para guardar y buscar controles |
| `ChildPort`, `CareServicePort`, … | Interfaces | Dominio | Contrato para consultar otros módulos |
| `ChildRef`, `CareServiceRef`, … | Objetos de valor | Dominio | Solo los datos que el módulo necesita de otro |
| `AsyncpgChildControlRepository` | Repositorio | Infraestructura | Implementa el puerto con PostgreSQL |
| `ChildAdapter`, `CareServiceAdapter`, … | Adaptadores | Infraestructura | Consultan otros módulos y traducen a `*Ref` |

### 2.3 Notación UML utilizada

| Símbolo | Significado | Ejemplo |
|---|---|---|
| Línea continua con flecha | Asociación: guarda una referencia | `ChildControlCreateUseCase` → `ChildControlRepositoryPort` |
| Línea discontinua con flecha | Dependencia: la usa | handler ⇢ `ChildControlService` |
| Línea discontinua con triángulo hueco | Realización: implementa una interfaz | `AsyncpgChildControlRepository` ▷ `ChildControlRepositoryPort` |
| Rombo relleno | Composición | `ChildControlService` ◆ `ChildControlCreateUseCase` |
| «interface» | Interfaz (`Protocol` en Python) | `ChildPort` |

---

## 3. Diagrama de secuencia: registrar control CRED

```mermaid
sequenceDiagram
    autonumber
    actor PS as Personal de salud
    participant MW as AuthMiddleware
    participant H as Handler
    participant UC as ChildControlCreateUseCase
    participant CA as ChildAdapter
    participant SA as CareServiceAdapter
    participant R as AsyncpgChildControlRepository
    participant DB as PostgreSQL

    PS->>MW: POST /api/child-controls + cookie JWT
    MW->>MW: valida el JWT y la sesión activa, y fija el contexto de auditoría
    MW->>H: require_permission("child_control.post") ✔
    H->>UC: execute(ChildControl)
    UC->>CA: get_by_id(child_id)
    CA-->>UC: ChildRef
    UC->>SA: get_by_id(service_id)
    SA-->>UC: CareServiceRef
    alt regla incumplida
        UC-->>H: ChildControlValidationError
        H-->>PS: 400 Bad Request
    else datos válidos
        UC->>R: create(control)
        R->>DB: INSERT INTO child_controls
        DB->>DB: trigger de auditoría (audit_log)
        alt control duplicado
            DB-->>R: UniqueViolationError
            R-->>H: AlreadyExistsError (traducido)
            H-->>PS: 409 Conflict
        else
            DB-->>R: fila
            R-->>UC: ChildControl
            UC-->>H: ChildControl
            H-->>PS: 201 Created
        end
    end
```

---

## 4. Patrones y principios que se ven en este nivel

| Elemento del diagrama | Patrón o principio |
|---|---|
| `ChildAdapter`, `CareServiceAdapter` | Patrón **Adapter** |
| `ChildControlRepositoryPort` y `AsyncpgChildControlRepository` | Patrón **Repository** |
| `translate_db_errors` sobre el repositorio | Patrón **Decorator** |
| `ChildControlService` | Patrón **Facade** |
| El caso de uso recibe interfaces, no clases concretas | Principio de **inversión de dependencias** (SOLID) |
| `ChildRef` solo con 3 datos del niño | Principio de **segregación de interfaces** (SOLID) |

> Detalle en [Patrones de diseño](../03-diseno-de-software/02-patrones-de-diseno.md) y [Principios de diseño](../03-diseno-de-software/03-principios-de-diseno.md).
