# Decisiones arquitectónicas (ADR)

| ID | Decisión arquitectónica | Driver relacionado | Justificación | Resultado |
|---|---|---|---|---|
| ADR-001 | **Monolito modular** | DA04 – Mantenibilidad; DA06 – Simple de desplegar | Un solo backend es fácil de desplegar y de probar; dividirlo en módulos por dominio mantiene el orden. | 19 módulos en `srs-back/src/modules/`, desplegados juntos en una sola API. |
| ADR-002 | **Clean Architecture dentro de cada módulo** | DA04 – Mantenibilidad | Separar las reglas del negocio de FastAPI y de PostgreSQL. | Capas `domain`, `application`, `infrastructure` y `presentation` en cada módulo. |
| ADR-003 | **Comunicación entre módulos mediante puertos y adaptadores** | DA04 – Mantenibilidad | Un módulo no debe depender del código interno de otro. | Cada módulo declara en su dominio lo que necesita de otro (por ejemplo `ChildPort`) y un adaptador lo implementa. |
| ADR-004 | **Autenticación con JWT en cookie `httpOnly` y sesiones en base de datos** | DA01 – Seguridad | El token no es accesible desde JavaScript y cada sesión puede revocarse en el servidor. | Módulos `auth` y `session`, y middleware `AuthMiddleware`. |
| ADR-005 | **Control de acceso por permisos (RBAC)** | DA01 – Seguridad | Los permisos por operación (`child.post`, `claim.put`…) son más precisos que verificar solo el rol. | Catálogo de permisos, `require_permission()` en cada ruta y roles editables. |
| ADR-006 | **Auditoría con triggers de PostgreSQL** | DA02 – Trazabilidad | La auditoría no depende de que cada programador se acuerde de registrarla. | Tabla `audit_log` y contexto de auditoría (usuario, dispositivo, IP) enviado en cada conexión. |
| ADR-007 | **Reglas validadas en el negocio y reforzadas en la base de datos** | DA03 – Integridad | Doble barrera: mensajes claros desde la aplicación y garantía final en la base de datos. | Validaciones en `application/` y triggers (`check_child_control_dates`, `check_claim_evidence_limit`). |
| ADR-008 | **Detección de riesgo por dispositivo** | DA01 – Seguridad | Detectar cuentas compartidas o robadas sin servicios externos. | Módulos `device` y `security_alert`; bloqueo por cuenta o por navegador. |

## Alternativas descartadas

| Decisión | Alternativa descartada | Motivo |
|---|---|---|
| Monolito modular | Microservicios | Equipo pequeño y plazo de un ciclo; los microservicios agregan red, despliegues y datos distribuidos sin un beneficio claro hoy. |
| Clean Architecture | Capas simples (controlador → servicio → repositorio) sin interfaces | Con capas simples el negocio queda atado a la base de datos y es difícil de probar sin ella. |
| JWT en cookie `httpOnly` | JWT en `localStorage` | En `localStorage` el token puede robarse con un ataque XSS. |

## Trabajo futuro

| Tema | Detalle |
|---|---|
| Integración con sistemas del MINSA | Si se integra con sistemas oficiales, bastaría con agregar un adaptador en `infrastructure/` sin modificar los casos de uso. |
| Almacenamiento de archivos en la nube | Reemplazar `LocalFileStorage` por un adaptador de almacenamiento en la nube que implemente el mismo `FileStoragePort`. |
| Notificaciones a los padres | Avisar por SMS o WhatsApp sobre controles próximos o vencidos. |
