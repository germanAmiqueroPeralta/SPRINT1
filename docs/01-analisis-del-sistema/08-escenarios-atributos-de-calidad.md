# Escenarios de atributos de calidad

## EQ-01. Seguridad: un padre intenta ver datos de otro niño

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad |
| **Fuente** | Padre o apoderado autenticado |
| **Estímulo** | Intenta registrar un reclamo o consultar información de un niño que no es su hijo, o entrar a una sección administrativa. |
| **Condición** | Sistema en funcionamiento normal; el padre tiene una sesión válida. |
| **Respuesta esperada** | El sistema rechaza la operación (403) y no revela datos del otro niño. |
| **Medida** | El 100 % de los intentos de las pruebas son rechazados, sin datos sensibles en la respuesta. |
| **Verificación** | Prueba realizada: el padre intentó reclamar sobre un niño ajeno y entrar a `/users`, y ambos intentos fueron rechazados (ver [verificación](../05-verificacion/01-verificacion-de-la-arquitectura.md)). |
| **Estado** | Verificado en pruebas. |

## EQ-02. Seguridad: sesión robada o dispositivo bloqueado

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad |
| **Fuente** | Administrador |
| **Estímulo** | Revoca una sesión o bloquea un dispositivo sospechoso. |
| **Condición** | El usuario afectado tiene una sesión abierta. |
| **Respuesta esperada** | La siguiente petición con esa sesión es rechazada (401) y el dispositivo bloqueado no puede volver a iniciar sesión. |
| **Medida** | Rechazo inmediato, desde la siguiente petición. |
| **Verificación** | El middleware valida en cada petición que la sesión siga activa en la base de datos. |
| **Estado** | Implementado; falta una prueba formal documentada. |

## EQ-03. Trazabilidad: cambio en los datos de un niño

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Trazabilidad |
| **Fuente** | Personal de salud |
| **Estímulo** | Modifica los datos de un niño o de un control. |
| **Condición** | Usuario autenticado. |
| **Respuesta esperada** | Se crea un registro de auditoría con usuario, dispositivo, IP, acción, tabla y valores anteriores y nuevos. |
| **Medida** | El 100 % de las operaciones INSERT, UPDATE y DELETE sobre las tablas auditadas quedan registradas. |
| **Verificación** | Consultar `/api/audit-logs` después de cada cambio. |
| **Estado** | Implementado con triggers de PostgreSQL. |

## EQ-04. Integridad: control con fechas inválidas

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Integridad de datos |
| **Fuente** | Personal de salud |
| **Estímulo** | Registra un control como realizado pero sin fecha, o con una fecha anterior al nacimiento del niño. |
| **Condición** | Funcionamiento normal. |
| **Respuesta esperada** | El sistema rechaza el control (400) con un mensaje claro y no guarda nada. |
| **Medida** | 0 controles inválidos guardados. |
| **Verificación** | Prueba realizada: el control "completado sin fecha" fue rechazado con 400. |
| **Estado** | Verificado en pruebas. |

## EQ-05. Mantenibilidad: cambiar una tecnología

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Mantenibilidad |
| **Fuente** | Equipo de desarrollo |
| **Estímulo** | Se decide guardar las evidencias en la nube en lugar del disco local. |
| **Condición** | Sistema organizado con Clean Architecture. |
| **Respuesta esperada** | Solo se crea un adaptador nuevo que implemente `FileStoragePort` y se cambia una línea en el contenedor del módulo. Los casos de uso no se modifican. |
| **Medida** | 0 archivos modificados en `domain/` y `application/`. |
| **Verificación** | Revisión del código: los casos de uso solo conocen `FileStoragePort`. |
| **Estado** | Diseño verificado; el cambio de tecnología todavía no se realizó. |

## EQ-06. Disponibilidad: horario de atención

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Disponibilidad |
| **Fuente** | Personal de salud y padres |
| **Estímulo** | Usan el sistema durante el horario de atención. |
| **Condición** | Sistema publicado en un servidor. |
| **Respuesta esperada** | Las operaciones están disponibles; si la base de datos falla, la API responde con un error controlado (500) y sin exponer detalles internos. |
| **Medida** | **Propuesta por validar:** 99 % de disponibilidad en el horario de atención (de lunes a sábado, de 7:00 a 19:00). |
| **Verificación** | Monitoreo del servidor una vez desplegado. |
| **Estado** | Pendiente de despliegue. |

## Resumen de escenarios

| Código | Atributo | Qué se busca comprobar | Estado |
|---|---|---|---|
| EQ-01 | Seguridad | Que un padre solo acceda a sus propios hijos | Verificado |
| EQ-02 | Seguridad | Que una sesión revocada deje de funcionar | Implementado |
| EQ-03 | Trazabilidad | Que todo cambio quede auditado | Implementado |
| EQ-04 | Integridad | Que no se guarden controles inválidos | Verificado |
| EQ-05 | Mantenibilidad | Que cambiar una tecnología no afecte al negocio | Diseño verificado |
| EQ-06 | Disponibilidad | Que el sistema esté disponible en horario de atención | Pendiente |
