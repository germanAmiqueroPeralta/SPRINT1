# Atributos de calidad

| ID | Atributo de calidad | Escenario de calidad |
|---|---|---|
| AC01 | **Rendimiento** | Las consultas del padrón de niños, de los controles CRED y del portal familiar deben responder rápidamente incluso cuando muchos coordinadores y padres usan el sistema al mismo tiempo. La carga nocturna de atenciones no debe volver lento el sistema para los usuarios. |
| AC02 | **Disponibilidad** | El sistema debe permanecer disponible durante la jornada de atención de los establecimientos, para que el personal registre controles y los padres consulten el portal. Si RENIEC o la base externa de atenciones no responden, SICNA debe seguir funcionando: se permite el registro manual y la carga se reintenta después. |
| AC03 | **Escalabilidad** | El sistema debe poder soportar el crecimiento de niños, padres, establecimientos y micro redes registrados, y del volumen de atenciones que se cargan, sin afectar significativamente su funcionamiento. |
| AC04 | **Seguridad** | Los datos personales y de salud de los niños y las cuentas de los usuarios deben estar protegidos frente a accesos no autorizados. Solo acceden usuarios autenticados con el permiso adecuado, un padre solo ve a sus propios hijos, el coordinador no administra la plataforma y todo cambio queda registrado en la auditoría (Ley N.° 29733). |
| AC05 | **Mantenibilidad** | El sistema debe estar organizado en módulos y capas de manera que permita cambiar reglas del negocio, corregir errores o reemplazar una tecnología (base de datos, proveedor de identidad, almacenamiento) sin afectar innecesariamente otras funcionalidades. |
