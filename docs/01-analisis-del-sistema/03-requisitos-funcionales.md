# Requisitos funcionales

## Requisitos funcionales

| ID | Requisito funcional |
|---|---|
| RF01 | El sistema debe permitir iniciar y cerrar sesión con usuario y contraseña. |
| RF02 | El sistema debe permitir registrar, consultar, actualizar y desactivar niños. |
| RF03 | El sistema debe consultar la API de RENIEC con un DNI y devolver los datos de identidad para autocompletar el registro de niños y padres. |
| RF04 | El sistema debe permitir registrar padres o apoderados, vincularlos con sus hijos indicando el parentesco y, opcionalmente, crearles una cuenta del portal. |
| RF05 | El sistema debe permitir gestionar paquetes de atención y prestaciones con su rango de edad (catálogo SISNAT). |
| RF06 | El sistema debe permitir registrar y actualizar los controles de cada niño (pendiente, realizado, vencido, no aplica), validando que correspondan a un niño activo, a una prestación activa y a fechas coherentes con su nacimiento. |
| RF07 | El sistema debe leer periódicamente las atenciones de la base de datos externa y actualizar los controles CRED correspondientes. |
| RF08 | El sistema debe permitir gestionar usuarios, roles y permisos. |
| RF09 | El padre o apoderado debe poder consultar solo a sus propios hijos y sus controles desde el portal. |
| RF10 | El padre o apoderado debe poder presentar reclamos y adjuntar hasta 5 evidencias (JPG, PNG o PDF de hasta 10 MB). |
| RF11 | El coordinador debe poder revisar, responder, resolver o rechazar reclamos. |
| RF12 | El sistema debe registrar en una auditoría todo cambio sobre los datos (quién, qué, cuándo y desde dónde). |

---

## Relación entre historias de usuario y requisitos funcionales

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU01 Registrar niños | RF02 |
| HU02 Autocompletar con RENIEC | RF03 |
| HU03 Registrar y vincular padres | RF04 |
| HU04 Registrar controles CRED | RF06 |
| HU05 Carga automática de atenciones | RF07 |
| HU06 Resultado de la carga | RF07 |
| HU07 Gestionar catálogo | RF05 |
| HU08 Gestionar organización y accesos | RF08 |
| HU09 Portal familiar | RF01, RF09 |
| HU10 Presentar reclamos | RF10 |
| HU11 Atender reclamos | RF11 |
| HU12 Seguridad de cuentas | RF01, RF08 |
| HU13 Auditoría | RF12 |
