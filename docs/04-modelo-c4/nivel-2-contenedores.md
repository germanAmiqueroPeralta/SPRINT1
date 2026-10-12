# Nivel 2 · Diagrama de contenedores

> **Proyecto:** SICNA · **Modelo:** C4 · Nivel 2 de 4

---

## 1. Objetivo

Abrir la caja «SICNA» del nivel 1 y mostrar los **contenedores**: las aplicaciones y los almacenes de datos que se ejecutan por separado, su tecnología y cómo se comunican.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿De qué piezas ejecutables se compone el sistema y con qué tecnología? |
| Audiencia | Arquitectos y desarrolladores |
| Qué es un contenedor | Algo que se ejecuta o guarda datos de forma independiente (no es lo mismo que un contenedor Docker) |
| Nivel anterior | [Nivel 1 · Contexto](nivel-1-contexto.md) |
| Siguiente nivel | [Nivel 3 · Componentes](nivel-3-componentes.md) |

---

## 2. Diagrama

```mermaid
flowchart TB
    AG["👤 Administrador general"]
    AL["👤 Administrador limitado"]
    PA["👤 Padre / apoderado"]

    subgraph SICNA["SICNA · límite del sistema"]
        direction TB
        Web["<b>Aplicación web</b><br/><i>[Next.js 16 + React 19]</i><br/>Panel del administrador general,<br/>panel del administrador limitado<br/>y portal familiar"]
        API["<b>API SICNA</b><br/><i>[Python + FastAPI]</i><br/>Monolito modular:<br/>reglas de negocio, seguridad y permisos"]
        IMP["<b>Importador de atenciones</b><br/><i>[Python · proceso programado]</i><br/>Carga nocturna o a pedido;<br/>usa los mismos módulos que la API"]
        BD[("<b>Base de datos</b><br/><i>[PostgreSQL]</i><br/>Datos del negocio, sesiones,<br/>auditoría y triggers")]
        FS[("<b>Archivos</b><br/><i>[Sistema de archivos]</i><br/>Evidencias de reclamos")]
    end

    RENIEC["<b>API de RENIEC</b><br/><i>[Sistema externo]</i>"]
    BDEXT[("<b>BD de atenciones</b><br/><i>[Sistema externo]</i>")]

    AG & AL & PA -->|"Usa<br/>[HTTPS]"| Web
    Web -->|"Llama a la API<br/>[HTTP/JSON · cookie httpOnly con JWT]"| API
    AG -.->|"Inicia una carga<br/>desde su panel"| Web
    API -->|"Lee y escribe<br/>[SQL · asyncpg]"| BD
    API -->|"Guarda y lee evidencias"| FS
    API -->|"Consulta DNI<br/>[HTTPS/JSON · 5 s máx.]"| RENIEC
    API -.->|"Ordena una carga manual"| IMP
    IMP -->|"Lee atenciones nuevas<br/>[SQL · solo lectura]"| BDEXT
    IMP -->|"Actualiza controles y<br/>registra la carga [SQL]"| BD

    classDef persona fill:#08427b,color:#fff,stroke:#052e56
    classDef cont fill:#438dd5,color:#fff,stroke:#2e6295
    classDef ext fill:#999,color:#fff,stroke:#6b6b6b
    class AG,AL,PA persona
    class Web,API,IMP,BD,FS cont
    class RENIEC,BDEXT ext
```

---

## 3. Contenedores

| Contenedor | Tecnología | Responsabilidad | Datos que maneja | Estado |
|---|---|---|---|---|
| **Aplicación web** | Next.js 16, React 19, Tailwind 4 | Tres interfaces, una por tipo de usuario. Sus *server actions* llaman a la API | Ninguno persistente; reenvía la cookie de sesión | Implementado |
| **API SICNA** | Python, FastAPI, asyncpg | Expone la API REST, aplica las reglas de negocio, la autenticación y los permisos. Es un **monolito modular** | Ninguno propio; usa la base de datos y los archivos | Implementado |
| **Importador de atenciones** | Python (mismo código que la API) | Lee la BD externa, empareja cada atención con un niño y una prestación, actualiza los controles y registra el resultado | Atenciones externas en memoria durante la carga | Diseñado |
| **Base de datos** | PostgreSQL | Guarda la información y ejecuta la auditoría y las reglas de integridad con triggers | Usuarios, niños, padres, catálogo, controles, reclamos, sesiones, dispositivos, alertas, auditoría e historial de cargas | Implementado |
| **Archivos** | Sistema de archivos (`uploads/`) | Guarda las evidencias de los reclamos | JPG, PNG y PDF | Implementado |

---

## 4. Relaciones

| Origen | Destino | Descripción | Protocolo |
|---|---|---|---|
| Usuarios | Aplicación web | Usan el sistema desde el navegador | HTTPS |
| Aplicación web | API SICNA | Llama a la API | HTTP/JSON con cookie `access_token` |
| API SICNA | Base de datos | Lee y escribe datos; envía el contexto de auditoría | SQL sobre TCP |
| API SICNA | Archivos | Guarda, lee y borra evidencias | Acceso a disco |
| API SICNA | API de RENIEC | Consulta datos de identidad por DNI | HTTPS/JSON, tiempo límite de 5 s |
| API SICNA | Importador | El administrador general ordena una carga manual | Llamada interna |
| Importador | BD de atenciones | Lee las atenciones nuevas | SQL, usuario de solo lectura |
| Importador | Base de datos | Actualiza los controles y guarda el resultado de la carga | SQL sobre TCP |

---

## 5. Decisiones arquitectónicas reflejadas

| Decisión | Alternativa descartada | Motivo |
|---|---|---|
| Un único backend (monolito modular) | Microservicios | Equipo pequeño y plazo de un ciclo |
| Importador como proceso aparte que reutiliza los módulos | Hacer la carga dentro de una petición HTTP | Una carga larga no debe bloquear ni hacer esperar a los usuarios |
| Frontend separado del backend (Next.js + API REST) | Páginas generadas por el backend | Cada parte evoluciona por separado |
| PostgreSQL | Base de datos NoSQL | Los datos son relacionales y se necesitan transacciones, restricciones y triggers |
| JWT en cookie `httpOnly` + sesión en base de datos | JWT en `localStorage` | El token no es accesible desde JavaScript y la sesión se puede revocar |
| Solo la API consulta RENIEC | Consultar RENIEC desde el navegador | Las credenciales quedan en el servidor |

---

## 6. Seguridad entre contenedores

| Comunicación | Medida |
|---|---|
| Navegador → aplicación web | HTTPS en producción (`COOKIE_SECURE=true`) |
| Aplicación web → API | JWT en cookie `httpOnly` y `SameSite=Lax`; en cada petición se verifica que la sesión siga activa |
| API → base de datos | Credenciales en variables de entorno (`.env`), que no se suben al repositorio |
| API → RENIEC | Token de RENIEC en variables de entorno; cada consulta queda registrada |
| Importador → BD externa | Usuario de base de datos con permiso **solo de lectura** |
| Contraseñas | Guardadas con Argon2, nunca en texto plano |
