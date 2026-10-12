# Drivers arquitectónicos

Los **drivers arquitectónicos** son los requisitos que más influyen en la forma del sistema.

| ID | Driver arquitectónico | Origen | ¿Por qué influye en la arquitectura? |
|---|---|---|---|
| DA01 | Proteger los datos de menores y controlar quién accede a qué, con dos niveles de administrador. | AC04 – Seguridad; RC04 | Obliga a tener autenticación, sesiones revocables, permisos por operación (no solo por rol) y aislamiento de datos en el portal familiar. |
| DA02 | Registrar quién hizo cada cambio, sea una persona o el importador. | AC04 – Seguridad | El contexto (usuario o proceso, dispositivo, IP) debe llegar hasta la auditoría de la base de datos. |
| DA03 | Evitar datos inconsistentes de controles, atenciones y reclamos. | RF08; RC08 | Exige validar las reglas en el negocio y reforzarlas en la base de datos. |
| DA04 | Integrarse con RENIEC y con la base externa de atenciones. | AC05 – Mantenibilidad; RC05; RC06 | Obliga a aislar cada sistema externo detrás de una interfaz propia, para que su formato no contamine el negocio. |
| DA05 | Seguir funcionando si un sistema externo falla. | AC02 – Disponibilidad | Las integraciones deben tener tiempo límite, reintentos y una alternativa manual. La carga de atenciones debe ejecutarse fuera de las peticiones de los usuarios. |
| DA06 | Poder cambiar reglas o tecnologías sin romper otros módulos. | AC05 – Mantenibilidad | Influye en la separación por módulos y en la dirección de las dependencias internas. |
| DA07 | Usar una API REST entre frontend y backend. | RC02 – API REST | Limita la forma de comunicación entre las partes del sistema. |
| DA08 | Ser simple de desplegar con un equipo pequeño. | RC09 – Equipo y plazo | Favorece un único backend desplegable en lugar de muchos servicios separados. |
