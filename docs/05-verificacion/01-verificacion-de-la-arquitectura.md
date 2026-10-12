# Verificación de la arquitectura

## Objetivo

Comprobar con evidencia, y no solo con diagramas, que el backend cumple Clean Architecture y que el cambio de arquitectura **no alteró el comportamiento** del sistema.

## Contexto

El backend se organizaba antes en capas simples (`handlers → services → repositories`). Se migraron los 19 módulos a Clean Architecture y luego se verificó el resultado de tres maneras.

---

## 1. Regla de dependencia (análisis estático)

Se buscaron automáticamente, en todos los archivos de `domain/` y `application/` de los 19 módulos, imports prohibidos:

| Se buscó | Resultado |
|---|---|
| `fastapi`, `starlette`, `pydantic` | 0 apariciones |
| `asyncpg`, `jwt` | 0 apariciones |
| `core.db`, `core.security`, `core.storage`, `core.http`, `core.config`, `core.logger` | 0 apariciones |
| Capas `infrastructure/`, `presentation/` o contenedores de cualquier módulo | 0 apariciones |
| Capa `application/` de **otro** módulo | 0 apariciones |

**Conclusión:** el dominio y los casos de uso no dependen de ninguna tecnología.

---

## 2. Contrato de la API (OpenAPI)

FastAPI genera automáticamente la especificación OpenAPI de la API. Se comparó la especificación de la versión anterior con la de la nueva:

| Elemento | Antes | Después |
|---|---|---|
| Rutas | 46 | 46 |
| Operaciones (GET, POST, PUT, DELETE) | 89 | 89 |
| Parámetros y cuerpos de cada operación | — | Idénticos |
| Esquemas de datos | — | Idénticos |

**Conclusión:** el frontend no necesitó ningún cambio.

---

## 3. Comportamiento (pruebas contra la base de datos real)

Se ejecutaron **la versión anterior y la nueva al mismo tiempo**, conectadas a la misma base de datos, y se les enviaron las mismas peticiones. Luego se compararon el código HTTP, el mensaje y los datos de cada respuesta.

| Grupo de pruebas | Ejemplos | Resultado |
|---|---|---|
| Listados y consultas por id | Los 18 listados, consultas por id, ids inexistentes (404), ids inválidos (400) | Iguales |
| Filtros | Establecimientos por micro red, prestaciones por paquete, controles por niño | Iguales |
| Validaciones (400) | Control "completado" sin fecha, reclamo con fecha futura, paquete con edad mínima mayor que la máxima | Iguales |
| Conflictos (409) | Micro red, rol, permiso, código de padrón o DNI duplicados | Iguales |
| Autenticación | Login correcto, contraseña incorrecta, usuario inexistente, `/auth/me`, logout | Iguales |
| Portal familiar | Vista del padre, sus reclamos, reclamo sobre un niño ajeno (403), acceso a una sección administrativa (403) | Iguales |
| **Total** | **139 peticiones** | **139 iguales, 0 diferentes** |

Además se probó un ciclo completo de escritura en la versión nueva: crear, editar y borrar una micro red, un establecimiento, un paquete, una prestación y un permiso. Todo respondió correctamente y se comprobó en la base de datos que no quedaron registros de prueba.

---

## Resumen

| Verificación | Qué demuestra | Resultado |
|---|---|---|
| Regla de dependencia | Se cumple Clean Architecture | ✅ |
| Contrato OpenAPI | La API no cambió | ✅ |
| Comparación de respuestas | El comportamiento no cambió | ✅ 139 de 139 |
| Ciclo de escritura | Crear, editar y borrar funcionan con la nueva arquitectura | ✅ |

---

## Pendiente de verificar

Los módulos **diseñados** todavía no existen en el código, así que no forman parte de estas pruebas. Cuando se implementen, se verificarán así:

| Módulo | Prueba prevista |
|---|---|
| `identity` (RENIEC) | Probar con un adaptador falso: DNI encontrado (200), no encontrado (404), RENIEC caído o lento (503 en 5 s como máximo) |
| `attention_sync` (carga de atenciones) | Ejecutar la carga con datos de prueba que incluyan atenciones válidas, de niños inexistentes y con fechas inválidas; luego repetirla y comprobar que no se duplicaron controles |
| Ambos | Volver a ejecutar la búsqueda automática de la regla de dependencia sobre los módulos nuevos |
