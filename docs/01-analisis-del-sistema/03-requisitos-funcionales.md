# Requisitos funcionales

## Requisitos funcionales

| ID | Requisito funcional | Módulo del backend | Estado |
|---|---|---|---|
| RF01 | El sistema debe permitir iniciar y cerrar sesión con usuario y contraseña. | `auth`, `session` | Implementado |
| RF02 | El sistema debe permitir registrar, consultar, actualizar y desactivar niños. | `child` | Implementado |
| RF03 | El sistema debe consultar la API de RENIEC con un DNI y devolver los datos de identidad para autocompletar el registro de niños y padres. | `identity` | Diseñado |
| RF04 | El sistema debe permitir registrar padres o apoderados y, opcionalmente, crearles una cuenta del portal. | `guardian` | Implementado |
| RF05 | El sistema debe permitir vincular niños con sus padres o apoderados indicando el parentesco. | `child_guardian` | Implementado |
| RF06 | El sistema debe permitir gestionar paquetes de atención y prestaciones con su rango de edad. | `care_package`, `care_service` | Implementado |
| RF07 | El sistema debe permitir registrar y actualizar los controles de cada niño (pendiente, realizado, vencido, no aplica). | `child_control` | Implementado |
| RF08 | El sistema debe validar que el control corresponda a un niño activo, a una prestación activa y a fechas coherentes con su nacimiento. | `child_control` | Implementado |
| RF09 | El sistema debe leer periódicamente las atenciones de la base de datos externa y actualizar los controles CRED correspondientes. | `attention_sync` | Diseñado |
| RF10 | El sistema debe registrar el resultado de cada carga (fecha, atenciones leídas, aplicadas, omitidas y el motivo de cada omisión). | `attention_sync` | Diseñado |
| RF11 | El sistema debe permitir gestionar micro redes, establecimientos y asignar micro redes al personal. | `region`, `health_facility`, `user_region` | Implementado |
| RF12 | El sistema debe permitir gestionar usuarios, roles y permisos. | `user`, `role`, `permission` | Implementado |
| RF13 | El padre o apoderado debe poder consultar solo a sus propios hijos y sus controles desde el portal. | `portal` | Implementado |
| RF14 | El padre o apoderado debe poder presentar reclamos y adjuntar hasta 5 evidencias (JPG, PNG o PDF de hasta 10 MB). | `portal`, `claim`, `claim_evidence` | Implementado |
| RF15 | El administrador limitado debe poder revisar, responder, resolver o rechazar reclamos. | `claim` | Implementado |
| RF16 | El sistema debe registrar cada dispositivo y sesión, y permitir revocarlos o bloquearlos. | `device`, `session` | Implementado |
| RF17 | El sistema debe detectar actividad de riesgo (cuentas compartidas o muchos dispositivos) y generar alertas. | `security_alert` | Implementado |
| RF18 | El sistema debe registrar en una auditoría todo cambio sobre los datos (quién, qué, cuándo y desde dónde). | `audit_log` | Implementado |

> **Implementado:** existe en el código actual. **Diseñado:** está definido en esta arquitectura y pendiente de programar.

---

## Relación entre historias de usuario y requisitos funcionales

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU01 Registrar niños | RF02 |
| HU02 Autocompletar con RENIEC | RF03 |
| HU03 Registrar y vincular padres | RF04, RF05 |
| HU04 Registrar controles CRED | RF07, RF08 |
| HU05 Carga automática de atenciones | RF09 |
| HU06 Resultado de la carga | RF10 |
| HU07 Gestionar catálogo | RF06 |
| HU08 Gestionar organización y accesos | RF11, RF12 |
| HU09 Portal familiar | RF01, RF13 |
| HU10 Presentar reclamos | RF14 |
| HU11 Atender reclamos | RF15 |
| HU12 Seguridad de cuentas | RF16, RF17 |
| HU13 Auditoría | RF18 |
