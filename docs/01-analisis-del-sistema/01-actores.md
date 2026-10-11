# Actores

## Actores principales (personas)

| Actor | Rol en el sistema | ¿Qué necesita realizar? |
|---|---|---|
| **Administrador** | `admin` | Administrar toda la plataforma: usuarios, roles y permisos, micro redes y establecimientos, catálogo de paquetes y prestaciones, revisar la auditoría y la seguridad (sesiones, dispositivos y alertas de riesgo). |
| **Personal de salud** (coordinador) | `health_staff` | Registrar niños y a sus padres o apoderados, vincularlos, registrar y actualizar los controles CRED, y atender (responder) los reclamos de las familias. |
| **Padre, madre o apoderado** | `guardian` | Ingresar al portal familiar, consultar a sus hijos y sus controles, presentar reclamos y adjuntar evidencias (fotos o PDF). |

## Actores secundarios (sistemas)

| Actor | Tipo | ¿Qué hace? |
|---|---|---|
| **Base de datos PostgreSQL** | Almacén de datos | Guarda la información del negocio y ejecuta reglas de integridad y auditoría (triggers). |
| **Almacenamiento de archivos** | Almacén de archivos | Guarda las evidencias (JPG, PNG o PDF) que acompañan a los reclamos. |

> En esta versión SICNA **no se integra con sistemas externos de terceros**. El catálogo SISNAT de paquetes y prestaciones se carga como datos dentro del propio sistema. La integración con sistemas del MINSA queda como trabajo futuro (ver [decisiones arquitectónicas](07-decisiones-arquitectonicas.md)).

## Diferencia con un sistema de comercio

A diferencia de un marketplace, en SICNA **no hay pagos ni ventas**: el valor del sistema está en el **seguimiento oportuno de la salud del niño**, la **trazabilidad** de cada cambio y la **protección de los datos** de menores de edad.
