# Decisiones arquitectónicas (ADR)

| ID | Decisión arquitectónica | Driver relacionado | Justificación | Resultado |
|---|---|---|---|---|
| ADR-001 | **Monolito modular** | DA06 – Mantenibilidad; DA08 – Simple de desplegar | Un solo backend es fácil de desplegar y de probar; dividirlo en módulos por dominio mantiene el orden. | Módulos en `srs-back/src/modules/`, desplegados juntos en una sola API. |
| ADR-002 | **Clean Architecture dentro de cada módulo** | DA06 – Mantenibilidad | Separar las reglas del negocio de FastAPI, de PostgreSQL y de los sistemas externos. | Capas `domain`, `application`, `infrastructure` y `presentation` en cada módulo. |
| ADR-003 | **Comunicación entre módulos mediante puertos y adaptadores** | DA06 – Mantenibilidad | Un módulo no debe depender del código interno de otro. | Cada módulo declara en su dominio lo que necesita de otro (por ejemplo `ChildPort`) y un adaptador lo implementa. |
| ADR-004 | **Autenticación con JWT en cookie `httpOnly` y sesiones en base de datos** | DA01 – Seguridad | El token no es accesible desde JavaScript y cada sesión puede revocarse en el servidor. | Módulos `auth` y `session`, y middleware `AuthMiddleware`. |
| ADR-005 | **Control de acceso por permisos (RBAC) con dos niveles de administrador** | DA01 – Seguridad | El administrador general y el limitado comparten pantallas, pero no operaciones; los permisos por operación (`child.post`, `user.post`…) marcan la diferencia. | Catálogo de permisos, `require_permission()` en cada ruta y roles editables. |
| ADR-006 | **Auditoría con triggers de PostgreSQL** | DA02 – Trazabilidad | La auditoría no depende de que cada programador se acuerde de registrarla, y también cubre los cambios del importador. | Tabla `audit_log` y contexto de auditoría enviado en cada conexión. |
| ADR-007 | **Reglas validadas en el negocio y reforzadas en la base de datos** | DA03 – Integridad | Doble barrera: mensajes claros desde la aplicación y garantía final en la base de datos. | Validaciones en `application/` y triggers y restricciones en PostgreSQL. |
| ADR-008 | **Detección de riesgo por dispositivo** | DA01 – Seguridad | Detectar cuentas compartidas o robadas sin servicios externos. | Módulos `device` y `security_alert`; bloqueo por cuenta o por navegador. |
| ADR-009 | **RENIEC detrás de un puerto (`IdentityProviderPort`) y un adaptador (`ReniecApiAdapter`)** | DA04 – Interoperabilidad; DA05 – Tolerancia a fallos | El formato y las credenciales de RENIEC quedan en un solo archivo. Si RENIEC falla, el caso de uso devuelve "no disponible" y se permite el registro manual. | Nuevo módulo `identity`. Tiempo límite, caché de consultas y registro de cada consulta. |
| ADR-010 | **Carga de atenciones como proceso programado e idempotente** | DA04 – Interoperabilidad; DA05 – Disponibilidad | Leer la base externa puede tardar; hacerlo dentro de una petición bloquearía a los usuarios. Si la carga se repite, no debe duplicar controles. | Nuevo módulo `attention_sync` con el **Importador de atenciones**, que corre de forma programada o a pedido del administrador general. |
| ADR-011 | **Acceso de solo lectura a la base externa mediante un adaptador** | DA04; RC06 | SICNA no debe modificar la base de la red de salud, y su estructura no debe filtrarse al dominio. | Puerto `AttentionSourcePort` y adaptador `ExternalDbAttentionSource`, que traduce cada fila a una `ExternalAttention`. |

## Alternativas descartadas

| Decisión | Alternativa descartada | Motivo |
|---|---|---|
| Monolito modular | Microservicios | Equipo pequeño y plazo de un ciclo; los microservicios agregan red, varios despliegues y datos distribuidos sin un beneficio claro hoy. |
| Clean Architecture | Capas simples (controlador → servicio → repositorio) sin interfaces | Con capas simples el negocio queda atado a la base de datos y es difícil de probar sin ella. |
| JWT en cookie `httpOnly` | JWT en `localStorage` | En `localStorage` el token puede robarse con un ataque XSS. |
| Importador programado | Consultar la base externa en cada pantalla | Haría que SICNA dependa en todo momento de la base externa y la volvería lenta. |
| Importador programado | Que la base externa escriba directamente en la base de SICNA | Se saltaría las reglas del negocio y la auditoría. |
| Consulta a RENIEC desde el backend | Consultar RENIEC desde el navegador | Expondría las credenciales de RENIEC al usuario. |

## Trabajo futuro

| Tema | Detalle |
|---|---|
| Almacenamiento de archivos en la nube | Reemplazar `LocalFileStorage` por un adaptador de nube que implemente el mismo `FileStoragePort`. |
| Notificaciones a los padres | Avisar por SMS o WhatsApp sobre controles próximos o vencidos. |
