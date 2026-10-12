# Actores

## Actores principales (usuarios)

SICNA tiene **3 tipos de usuario**. Cada uno corresponde a un rol del sistema con sus propios permisos.

| Actor | Rol en el sistema | ¿Qué necesita realizar? |
|---|---|---|
| **Administrador general** | `admin` | Tiene acceso total. Administra usuarios, roles y permisos, micro redes y establecimientos, y el catálogo de paquetes y prestaciones. Revisa la auditoría y la seguridad (sesiones, dispositivos y alertas de riesgo) y supervisa la carga de atenciones desde la base de datos externa. |
| **Coordinador** | `health_staff` | Tiene acceso restringido a la atención del niño. Registra niños y padres, los vincula, registra y corrige controles CRED y responde los reclamos de las familias. **No puede** gestionar usuarios, roles, permisos, el catálogo ni la seguridad. |
| **Padre, madre o apoderado** | `guardian` | Usa el portal familiar: consulta a sus hijos y sus controles, presenta reclamos y adjunta evidencias (fotos o PDF). Solo ve la información de sus propios hijos. |

### Comparación de permisos

| Funcionalidad | Administrador general | Coordinador | Padre / apoderado |
|---|:---:|:---:|:---:|
| Usuarios, roles y permisos | ✅ | — | — |
| Micro redes y establecimientos | ✅ | Solo consulta | — |
| Catálogo de paquetes y prestaciones | ✅ | Solo consulta | Solo consulta (portal) |
| Niños, padres y vínculos | ✅ | ✅ | Solo sus hijos |
| Controles CRED | ✅ | ✅ | Solo los de sus hijos |
| Reclamos | ✅ | Revisar y responder | Presentar y adjuntar evidencias |
| Consulta de DNI en RENIEC | ✅ | ✅ | — |
| Carga de atenciones desde la BD externa | ✅ (ejecutar y revisar) | Solo consulta del resultado | — |
| Sesiones, dispositivos, alertas y auditoría | ✅ | — | — |

> Los permisos no están fijos en el código: se asignan a cada rol desde la base de datos (RBAC), así que el administrador general puede ampliar o reducir lo que hace el coordinador.

## Actores secundarios (sistemas externos)

| Actor | Tipo | ¿Qué hace? |
|---|---|---|
| **API de RENIEC** | Servicio web externo | Devuelve los datos de identidad de una persona a partir de su DNI (nombres, apellidos, fecha de nacimiento). SICNA lo usa para validar y autocompletar los datos de niños y padres. |
| **Base de datos externa de atenciones** | Base de datos de la red de salud | Contiene las atenciones registradas en los establecimientos. SICNA la **lee periódicamente** para actualizar los controles CRED de cada niño. |

## Actores secundarios (internos)

| Actor | Tipo | ¿Qué hace? |
|---|---|---|
| **Importador de atenciones** | Proceso programado | Se ejecuta automáticamente (por ejemplo, cada noche) o cuando el administrador general lo inicia. Lee la base externa y actualiza los controles. |
| **Base de datos PostgreSQL** | Almacén de datos | Guarda la información del negocio y ejecuta las reglas de integridad y la auditoría (triggers). |
| **Almacenamiento de archivos** | Almacén de archivos | Guarda las evidencias de los reclamos. |
