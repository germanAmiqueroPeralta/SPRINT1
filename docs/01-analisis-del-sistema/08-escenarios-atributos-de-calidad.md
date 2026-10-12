# Escenarios de atributos de calidad

## EQ-01. Seguridad: un padre intenta ver datos de otro niño

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad |
| **Fuente** | Padre o apoderado autenticado |
| **Estímulo** | Intenta registrar un reclamo sobre un niño que no es su hijo o entrar a una sección administrativa. |
| **Condición** | El padre tiene una sesión válida. |
| **Respuesta esperada** | El sistema rechaza la operación (403) y no revela datos del otro niño. |
| **Medida** | El 100 % de los intentos de las pruebas son rechazados, sin datos sensibles en la respuesta. |
| **Verificación** | Prueba realizada: el padre intentó reclamar sobre un niño ajeno y entrar a `/users`, y ambos intentos fueron rechazados (ver [verificación](../05-verificacion/01-verificacion-de-la-arquitectura.md)). |
| **Estado** | Verificado en pruebas. |

## EQ-02. Seguridad: el administrador limitado intenta una operación del general

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad |
| **Fuente** | Administrador limitado autenticado |
| **Estímulo** | Intenta crear un usuario, cambiar los permisos de un rol o bloquear un dispositivo. |
| **Condición** | Su rol no tiene los permisos `user.post`, `role.put` ni `device.put`. |
| **Respuesta esperada** | El sistema responde 403 indicando el permiso que falta y no ejecuta el cambio. |
| **Medida** | El 100 % de las operaciones administrativas son rechazadas para este rol. |
| **Verificación** | `require_permission()` se ejecuta en cada ruta antes del caso de uso. |
| **Estado** | Implementado. |

## EQ-03. Interoperabilidad y tolerancia a fallos: RENIEC no responde

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Disponibilidad y tolerancia a fallos |
| **Fuente** | Administrador limitado |
| **Estímulo** | Consulta un DNI mientras la API de RENIEC está caída o tarda demasiado. |
| **Condición** | RENIEC no responde dentro del tiempo límite. |
| **Respuesta esperada** | SICNA responde "servicio de identidad no disponible" y permite completar el registro manualmente. El resto del sistema sigue funcionando. |
| **Medida** | **Propuesta por validar:** respuesta en 5 segundos como máximo, aunque RENIEC no conteste. |
| **Verificación** | Simular la caída con un adaptador falso que no responda. |
| **Estado** | Diseñado. |

## EQ-04. Integridad: carga de atenciones con datos que no coinciden

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Integridad de datos |
| **Fuente** | Importador de atenciones |
| **Estímulo** | La base externa trae atenciones de un niño que no está en SICNA, con un código de prestación desconocido o con una fecha anterior al nacimiento. |
| **Condición** | Carga programada nocturna. |
| **Respuesta esperada** | Esas atenciones se **omiten**, se registra el motivo de cada una y las demás se aplican normalmente. |
| **Medida** | 0 controles inválidos guardados; el 100 % de las omisiones tiene un motivo registrado. |
| **Verificación** | Ejecutar la carga con un juego de datos de prueba que incluya casos inválidos. |
| **Estado** | Diseñado. |

## EQ-05. Integridad: la carga se ejecuta dos veces

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Integridad de datos |
| **Fuente** | Administrador general |
| **Estímulo** | Ejecuta la carga manualmente después de la carga automática del mismo día. |
| **Condición** | Las atenciones ya fueron aplicadas. |
| **Respuesta esperada** | No se duplica ningún control (la carga es **idempotente**). |
| **Medida** | 0 controles duplicados. |
| **Verificación** | La base ya impide duplicados con la restricción `child_controls_unique (child_id, service_id)`; el importador actualiza en lugar de insertar. |
| **Estado** | Restricción implementada; importador diseñado. |

## EQ-06. Trazabilidad: cambios hechos por personas y por el importador

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Trazabilidad |
| **Fuente** | Administrador limitado o importador de atenciones |
| **Estímulo** | Se modifica un niño o un control. |
| **Condición** | Funcionamiento normal. |
| **Respuesta esperada** | Se crea un registro de auditoría con el autor, la acción, la tabla y los valores anteriores y nuevos. |
| **Medida** | El 100 % de las operaciones sobre las tablas auditadas quedan registradas. |
| **Verificación** | Consultar `/api/audit-logs` después de cada cambio. |
| **Estado** | Implementado con triggers de PostgreSQL. |

## EQ-07. Mantenibilidad: cambiar el proveedor de identidad

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Mantenibilidad |
| **Fuente** | Equipo de desarrollo |
| **Estímulo** | RENIEC cambia su API o se decide consultar a través de otro proveedor autorizado. |
| **Condición** | Sistema organizado con Clean Architecture. |
| **Respuesta esperada** | Solo se modifica o reemplaza `ReniecApiAdapter`; los casos de uso de niños y padres no cambian. |
| **Medida** | 0 archivos modificados en `domain/` y `application/`. |
| **Verificación** | Revisión del código: los casos de uso solo conocen `IdentityProviderPort`. |
| **Estado** | Diseñado. |

## Resumen de escenarios

| Código | Atributo | Qué se busca comprobar | Estado |
|---|---|---|---|
| EQ-01 | Seguridad | Que un padre solo acceda a sus propios hijos | Verificado |
| EQ-02 | Seguridad | Que el administrador limitado no administre la plataforma | Implementado |
| EQ-03 | Tolerancia a fallos | Que una caída de RENIEC no detenga el sistema | Diseñado |
| EQ-04 | Integridad | Que la carga no guarde atenciones inválidas | Diseñado |
| EQ-05 | Integridad | Que repetir la carga no duplique controles | Parcial |
| EQ-06 | Trazabilidad | Que todo cambio quede auditado | Implementado |
| EQ-07 | Mantenibilidad | Que cambiar el proveedor de identidad no afecte al negocio | Diseñado |
