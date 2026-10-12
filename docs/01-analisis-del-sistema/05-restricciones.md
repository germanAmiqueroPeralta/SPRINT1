# Restricciones

Restricciones que condicionan las decisiones de diseño y de arquitectura del sistema.

| ID | Restricción | Descripción |
|---|---|---|
| RC01 | **Aplicación web** | El sistema debe usarse desde un navegador web, en computadora y en celular. |
| RC02 | **API REST** | La comunicación entre el frontend y el backend debe hacerse mediante una API REST con JSON. |
| RC03 | **Tecnologías definidas** | El backend se desarrolla en Python con FastAPI, el frontend en Next.js y la base de datos es PostgreSQL. |
| RC04 | **Protección de datos personales** | Se manejan datos de menores de edad (nombre, DNI, fecha de nacimiento, condición al nacer). Deben protegerse conforme a la Ley N.° 29733, Ley de Protección de Datos Personales del Perú. |
| RC05 | **API de RENIEC** | La consulta de DNI se hace con las credenciales y las condiciones de uso que otorga RENIEC (o el proveedor autorizado). Las credenciales solo pueden estar en el servidor y cada consulta debe quedar registrada. |
| RC06 | **Base de datos externa de atenciones** | SICNA solo tiene acceso de **lectura** a esa base. No puede modificarla y debe adaptarse a su estructura (tablas, códigos de prestación y establecimiento). |
| RC07 | **Catálogo normativo** | Los paquetes y prestaciones deben seguir el catálogo SISNAT (rangos de edad en días y grupos de prestaciones). |
| RC08 | **Evidencias limitadas** | Las evidencias de un reclamo solo pueden ser JPG, PNG o PDF, de hasta 10 MB cada una y como máximo 5 por reclamo. |
| RC09 | **Equipo y plazo académicos** | El equipo es pequeño y el plazo es un ciclo académico, por lo que se necesita una solución simple de desplegar y de mantener. |
| RC10 | **Control de versiones** | El código fuente debe gestionarse con Git. |
