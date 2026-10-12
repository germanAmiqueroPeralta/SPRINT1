# Atributos de calidad

| ID | Atributo de calidad | Escenario de calidad |
|---|---|---|
| AC01 | **Seguridad** | Los datos personales y de salud de los niños solo deben ser accesibles por usuarios autenticados con el permiso adecuado. Un padre solo puede ver a sus propios hijos y el administrador limitado no puede administrar la plataforma. |
| AC02 | **Trazabilidad (auditabilidad)** | Todo cambio sobre los datos, incluidos los que hace el importador de atenciones, debe quedar registrado con el usuario o proceso, la fecha y los valores anteriores y nuevos. |
| AC03 | **Integridad de datos** | El sistema no debe aceptar información inconsistente: controles con fechas anteriores al nacimiento, atenciones de niños que no existen o más de 5 evidencias por reclamo. |
| AC04 | **Interoperabilidad** | El sistema debe integrarse con la API de RENIEC y con la base de datos externa de atenciones sin que sus formatos o cambios afecten las reglas del negocio. |
| AC05 | **Disponibilidad y tolerancia a fallos** | Si RENIEC o la base externa no responden, SICNA debe seguir funcionando: se permite el registro manual y la carga se reintenta después. |
| AC06 | **Mantenibilidad** | El sistema debe permitir cambiar una regla de negocio o una tecnología (base de datos, proveedor de identidad, almacenamiento) sin afectar otros módulos. |
| AC07 | **Usabilidad** | El portal familiar debe ser simple para padres sin experiencia técnica y usable desde el celular. |
