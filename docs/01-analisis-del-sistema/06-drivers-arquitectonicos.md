# Drivers arquitectónicos

Los **drivers arquitectónicos** son los requisitos que más influyen en la forma del sistema.

| ID | Driver arquitectónico | Origen | ¿Por qué influye en la arquitectura? |
|---|---|---|---|
| DA01 | Proteger los datos de menores y controlar quién accede a qué. | AC01 – Seguridad; RC04 – Protección de datos | Obliga a tener autenticación, sesiones revocables, permisos por operación y aislamiento de los datos en el portal familiar. |
| DA02 | Registrar quién hizo cada cambio. | AC02 – Trazabilidad | Influye en cómo se conectan la API y la base de datos: el contexto del usuario (usuario, dispositivo e IP) debe llegar hasta la auditoría. |
| DA03 | Evitar datos inconsistentes de controles y reclamos. | AC03 – Integridad; RC06 | Exige validar las reglas en el negocio y reforzarlas en la base de datos. |
| DA04 | Poder cambiar reglas o tecnologías sin romper otros módulos. | AC04 – Mantenibilidad | Influye en la separación por módulos y en la dirección de las dependencias internas. |
| DA05 | Usar una API REST entre frontend y backend. | RC02 – API REST | Limita la forma de comunicación entre las partes del sistema. |
| DA06 | Ser simple de desplegar con un equipo pequeño. | RC07 – Equipo y plazo | Favorece un único backend desplegable en lugar de muchos servicios separados. |
