# Patrones de diseño

## Objetivo

Documentar los patrones de diseño **que ya están aplicados** en el backend de SICNA: dónde se usan, qué problema resuelven y en qué archivos se pueden ver.

## Contexto

- **Estilo:** monolito modular (Python + FastAPI).
- **Patrón arquitectónico:** Clean Architecture.
- **Base:** diagrama de componentes (C4 nivel 3).

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Archivos de ejemplo |
|---|---|---|---|
| **Repository** | Todos los módulos → PostgreSQL | El negocio no debe conocer SQL ni asyncpg | `child_control_repository_port.py`, `child_control_repository.py` |
| **Adapter** | Comunicación entre módulos | Un módulo necesita datos de otro sin depender de su código interno | `child_control_adapters.py` (`ChildAdapter`, `CareServiceAdapter`…) |
| **Adapter** | Servicios técnicos | El negocio pide "guardar archivo" o "verificar contraseña" sin saber cómo se hace | `LocalFileStorage`, `Argon2PasswordVerifier`, `JwtTokenService` |
| **Adapter** *(diseñado)* | `identity` → API de RENIEC | El formato, las credenciales y los errores de RENIEC no deben llegar al negocio | `ReniecApiAdapter` implementa `IdentityProviderPort` |
| **Adapter** *(diseñado)* | `attention_sync` → BD externa de atenciones | La estructura de tablas de la red de salud es distinta a la de SICNA | `ExternalDbAttentionSource` implementa `AttentionSourcePort` |
| **Cache-Aside** *(diseñado)* | Consultas a RENIEC | No repetir la misma consulta de DNI ni gastar cuota del servicio | `IdentityLookupUseCase` consulta primero `IdentityCachePort` |
| **Batch Job programado** *(diseñado)* | Importador de atenciones | Procesar muchas atenciones sin bloquear las peticiones de los usuarios | `AttentionSyncUseCase` ejecutado por el importador |
| **Idempotencia** *(diseñado)* | Carga de atenciones | Repetir una carga no debe duplicar controles | Actualizar en lugar de insertar + restricción `child_controls_unique` |
| **Decorator** | Repositorios → errores de BD | Traducir los errores de asyncpg a errores del dominio sin repetir `try/except` en cada consulta | `core/db/db_errors.py` (`@translate_db_errors`) |
| **Facade** | Servicio de cada módulo | Ofrecer un solo punto de entrada a los 5 o más casos de uso del módulo | `child_control_service.py` (`ChildControlService`) |
| **Data Mapper** | Infraestructura | Convertir una fila de la base de datos en una entidad, fuera del dominio | `child_control_mapper.py` (`row_to_child_control`) |
| **Presenter** | Presentación | Convertir una entidad en JSON, fuera del dominio | `child_control_presenter.py` (`child_control_to_dict`) |
| **Inyección de dependencias + Composition Root** | Contenedor de cada módulo | Decidir en un solo lugar qué implementación concreta usa cada caso de uso | `child_control_container.py`, `Depends(get_child_control_service)` |
| **Parameter Object** | Casos de uso con muchas dependencias | Evitar constructores con 8 o 9 parámetros | `ChildControlRefs`, `AuthLoginDeps` |
| **Middleware (Interceptor)** | Todas las peticiones | Verificar el token y la sesión antes de llegar a cualquier módulo | `core/security/auth_middleware.py` |

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Repository | El caso de uso dice `create(control)` y el repositorio escribe el `INSERT`. |
| Adapter | El caso de uso pide un `ChildRef`; el adaptador consulta el módulo `child` y traduce su entidad `Child` a `ChildRef`. Igual con RENIEC: el caso de uso pide una `PersonIdentity` y el adaptador traduce la respuesta de RENIEC. |
| Decorator | `translate_db_errors` envuelve la función del repositorio: si PostgreSQL lanza `UniqueViolationError`, sale `AlreadyExistsError`. |
| Facade | Otros módulos y los handlers usan `childControlService.create(...)` sin conocer `ChildControlCreateUseCase`. |
| Data Mapper | `row_to_child_control(row)` arma el `ChildControl` a partir de la fila. |
| Presenter | `child_control_to_dict(control)` arma el JSON de respuesta. |
| Inyección de dependencias | El contenedor crea `AsyncpgChildControlRepository()` y se lo pasa al caso de uso; FastAPI entrega el servicio a la ruta con `Depends`. |

### Ejemplo: Decorator para los errores de base de datos

```python
# core/db/db_errors.py
def translate_db_errors(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except UniqueViolationError as err:
            raise AlreadyExistsError(err.constraint_name, str(err)) from err
        except ForeignKeyViolationError as err:
            raise ReferenceNotFoundError(err.constraint_name, str(err)) from err
        except CheckViolationError as err:
            raise InvalidDataError(err.constraint_name, str(err)) from err
    return wrapper
```

## Beneficio

- Cambiar la base de datos o el almacenamiento de archivos solo afecta al repositorio o al adaptador.
- Si el módulo `child` cambia por dentro, `child_control` no se entera: solo depende de `ChildRef`.
- Si RENIEC cambia su API o la red de salud cambia su base de datos, solo se modifica el adaptador correspondiente.
- Los 19 módulos usan los mismos patrones, así que todos se leen igual.

## Patrones previstos para siguientes iteraciones

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Observer | Controles → Notificaciones a los padres | Al implementar avisos de controles próximos o vencidos |
| Strategy | Cálculo del calendario de controles según la condición al nacer | Si las reglas por tipo de niño (prematuro, bajo peso) crecen |
| Decorator (caché) | Catálogo de paquetes y prestaciones | Si el catálogo se consulta mucho y conviene guardarlo en memoria |
