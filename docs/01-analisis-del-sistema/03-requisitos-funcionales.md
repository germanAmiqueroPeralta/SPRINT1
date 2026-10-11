# Requisitos funcionales

## Requisitos funcionales

| ID | Requisito funcional | Módulo del backend |
|---|---|---|
| RF01 | El sistema debe permitir iniciar y cerrar sesión con usuario y contraseña. | `auth`, `session` |
| RF02 | El sistema debe permitir registrar, consultar, actualizar y desactivar niños. | `child` |
| RF03 | El sistema debe permitir registrar padres o apoderados y, opcionalmente, crearles una cuenta del portal. | `guardian` |
| RF04 | El sistema debe permitir vincular niños con sus padres o apoderados indicando el parentesco. | `child_guardian` |
| RF05 | El sistema debe permitir gestionar paquetes de atención y prestaciones con su rango de edad. | `care_package`, `care_service` |
| RF06 | El sistema debe permitir registrar y actualizar los controles de cada niño (pendiente, realizado, vencido, no aplica). | `child_control` |
| RF07 | El sistema debe validar que el control corresponda a un niño activo, a una prestación activa y a fechas coherentes con su nacimiento. | `child_control` |
| RF08 | El sistema debe permitir gestionar micro redes, establecimientos de salud y asignar micro redes al personal. | `region`, `health_facility`, `user_region` |
| RF09 | El sistema debe permitir gestionar usuarios, roles y permisos. | `user`, `role`, `permission` |
| RF10 | El padre o apoderado debe poder consultar solo a sus propios hijos y sus controles desde el portal. | `portal` |
| RF11 | El padre o apoderado debe poder presentar reclamos y adjuntar hasta 5 evidencias (JPG, PNG o PDF de hasta 10 MB). | `portal`, `claim`, `claim_evidence` |
| RF12 | El personal de salud debe poder revisar, responder, resolver o rechazar reclamos. | `claim` |
| RF13 | El sistema debe registrar cada dispositivo y sesión, y permitir revocarlos o bloquearlos. | `device`, `session` |
| RF14 | El sistema debe detectar actividad de riesgo (cuentas compartidas o muchos dispositivos) y generar alertas. | `security_alert` |
| RF15 | El sistema debe registrar en una auditoría todo cambio sobre los datos (quién, qué, cuándo y desde dónde). | `audit_log` |

---

## Relación entre historias de usuario y requisitos funcionales

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU01 Registrar niños | RF02 |
| HU02 Registrar y vincular padres | RF03, RF04 |
| HU03 Registrar controles CRED | RF06, RF07 |
| HU04 Gestionar catálogo | RF05 |
| HU05 Gestionar organización y accesos | RF08, RF09 |
| HU06 Portal familiar | RF01, RF10 |
| HU07 Presentar reclamos | RF11 |
| HU08 Atender reclamos | RF12 |
| HU09 Seguridad de cuentas | RF13, RF14 |
| HU10 Auditoría | RF15 |
