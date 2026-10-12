# SICNA · Sistema de Control del Niño

## Integrantes

- *(completar)*

## Descripción

SICNA es un sistema web para que una red de salud haga el **seguimiento del control de crecimiento y desarrollo (CRED)** de los niños de su jurisdicción. Organiza el catálogo de paquetes y prestaciones de atención por edad (catálogo SISNAT), registra los controles de cada niño y ofrece a los padres un **portal familiar** donde ven el avance de sus hijos y pueden presentar reclamos con evidencias.

Además, SICNA **carga las atenciones** desde la base de datos de la red de salud para mantener los controles al día, y **consulta la API de RENIEC** para validar y autocompletar los datos de niños y padres a partir del DNI.

## Usuarios

| Usuario | Rol | Qué hace |
|---|---|---|
| **Administrador general** | `admin` | Acceso total: usuarios, permisos, catálogo, territorio, seguridad, auditoría y carga de atenciones |
| **Coordinador** | `health_staff` | Registra niños, padres y controles, y atiende reclamos |
| **Padre / apoderado** | `guardian` | Portal familiar: consulta a sus hijos y presenta reclamos |

## Caso de estudio

Red de salud con micro redes y establecimientos (centros y puestos de salud) que atienden a niños menores de 6 años. Las atenciones que registran los establecimientos se cargan en SICNA; el coordinador completa y corrige la información y los padres la consultan desde el portal.

## Curso

Arquitectura de Software

## Presentación

📊 [Ver la presentación de SICNA (Google Slides)](https://docs.google.com/presentation/d/1UOQCRtjdSHtLmeNqZJf_IEtyc8NsgQQi/edit?usp=drive_link&ouid=117345692440675930542&rtpof=true&sd=true)

## Tecnologías

| Parte | Tecnología |
|---|---|
| Frontend | Next.js 16 + React 19 + Tailwind 4 (`srs-front/`) |
| Backend | Python + FastAPI (`srs-back/`) |
| Base de datos | PostgreSQL |
| Archivos | Almacenamiento local (`uploads/`) para las evidencias de reclamos |
| Integraciones | API de RENIEC (HTTPS) y base de datos externa de atenciones (solo lectura) |

---

## Índice del informe

### 1. Análisis del sistema
1. [Actores](docs/01-analisis-del-sistema/01-actores.md)
2. [Historias de usuario](docs/01-analisis-del-sistema/02-historias-de-usuario.md)
3. [Requisitos funcionales](docs/01-analisis-del-sistema/03-requisitos-funcionales.md)
4. [Atributos de calidad](docs/01-analisis-del-sistema/04-atributos-de-calidad.md)
5. [Restricciones](docs/01-analisis-del-sistema/05-restricciones.md)
6. [Drivers arquitectónicos](docs/01-analisis-del-sistema/06-drivers-arquitectonicos.md)
7. [Decisiones arquitectónicas](docs/01-analisis-del-sistema/07-decisiones-arquitectonicas.md)
8. [Escenarios de atributos de calidad](docs/01-analisis-del-sistema/08-escenarios-atributos-de-calidad.md)

### 2. Arquitectura de software
1. [Arquitectura inicial](docs/02-arquitectura-de-software/01-arquitectura-inicial.md)
2. [Estilo arquitectónico](docs/02-arquitectura-de-software/02-estilo-arquitectonico.md)
3. [Patrón arquitectónico](docs/02-arquitectura-de-software/03-patron-arquitectonico.md)
4. [Componentes arquitectónicos](docs/02-arquitectura-de-software/04-componentes-arquitectonicos.md)

### 3. Diseño de software
1. [Diseño interno de módulos](docs/03-diseno-de-software/01-diseno-interno-de-modulos.md)
2. [Patrones de diseño](docs/03-diseno-de-software/02-patrones-de-diseno.md)
3. [Principios de diseño (SOLID)](docs/03-diseno-de-software/03-principios-de-diseno.md)

### 4. Modelo C4
1. [Nivel 1 · Contexto](docs/04-modelo-c4/nivel-1-contexto.md)
2. [Nivel 2 · Contenedores](docs/04-modelo-c4/nivel-2-contenedores.md)
3. [Nivel 3 · Componentes](docs/04-modelo-c4/nivel-3-componentes.md)
4. [Nivel 4 · Código](docs/04-modelo-c4/nivel-4-codigo.md)

---

## Resumen de la arquitectura

| Pregunta | Respuesta |
|---|---|
| **Estilo arquitectónico** | Monolito modular + capas |
| **Patrón arquitectónico** | Clean Architecture |
| **Módulos del backend** | 19 implementados + 2 diseñados (`identity` para RENIEC y `attention_sync` para la carga de atenciones) |
| **Usuarios** | 3: administrador general, coordinador y padre o apoderado |
| **Sistemas externos** | API de RENIEC y BD de atenciones de la red de salud |
| **Capas de cada módulo** | Dominio, Aplicación, Infraestructura y Presentación |

> Los diagramas están escritos en **Mermaid**. Se ven directamente en GitHub y en VS Code (con la extensión *Markdown Preview Mermaid Support*).
