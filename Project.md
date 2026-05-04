# Sistema de Gestión de Eventos Académicos

## 1. Visión General del Sistema

Plataforma web para organizar, gestionar y dar seguimiento a eventos académicos como cursos, jornadas, congresos, charlas y similares. El sistema permite a organizadores crear y administrar eventos, a los participantes inscribirse y obtener certificados, y a los disertantes gestionar su participación. La aplicación es accesible desde cualquier dispositivo a través de un navegador web.

## 2. Contexto del Producto

El software nace de la necesidad de centralizar y simplificar la organización de eventos académicos. Actualmente, muchos grupos y instituciones gestionan sus eventos de forma manual o dispersa entre múltiples herramientas. Esta plataforma unifica todo el ciclo de vida del evento:

- **Creación y configuración** de eventos con tipos personalizables, cupos y fechas límite.
- **Inscripción** de participantes de forma autónoma (web, enlace directo) o gestionada por el staff.
- **Gestión de roles** diferenciados con permisos y vistas específicas.
- **Acreditación** mediante código QR al momento del ingreso.
- **Encuestas de satisfacción** post-evento con evaluación por parte del disertante.
- **Generación automática de certificados** (asistencia, aprobación, disertante, autor).
- **Informes y agenda** para organizadores y participantes.
- **Notificaciones por email** de confirmación y recordatorio.

## 3. Módulos del Sistema

| # | Módulo | Descripción |
|---|---|---|
| 1 | **Autenticación y Gestión de Usuarios** | Registro, login, verificación de email, recuperación de contraseña, sesiones |
| 2 | **Gestión de Roles y Perfiles** | Roles: Administrador, Organizador, Disertante, Participante. Perfiles editables |
| 3 | **Gestión de Eventos** | CRUD de eventos, tipos configurables por admin, cupos min/max, fechas, filtros, listado público |
| 4 | **Inscripciones** | Inscripción autónoma, por enlace, por staff del evento. Lista de espera automática |
| 5 | **Certificados** | Generación de PDFs: asistencia, aprobación, disertante, autor |
| 6 | **Encuestas y Comentarios** | Formulario fijo de satisfacción post-evento, comentarios públicos por evento |
| 7 | **Informes y Agenda** | Cantidad de asistentes, agenda del evento, estadísticas de encuestas, listado de disertantes |
| 8 | **Acreditación y Notificaciones** | QR de acreditación, emails de confirmación de inscripción y recordatorio de evento |

## 4. Público Objetivo

| Rol | Descripción |
|---|---|
| **Administrador de Plataforma** | Gestiona usuarios, crea tipos de evento, supervisa el sistema |
| **Organizador** | Crea y configura eventos, gestiona inscripciones, acredita participantes, genera informes |
| **Disertante** | Participa como expositor en eventos, evalúa encuestas para otorgar aprobaciones |
| **Participante** | Se registra en la plataforma, se inscribe a eventos, completa encuestas, descarga certificados |

## 5. Stack Tecnológico

- **Backend:** Node.js + Express
- **Frontend:** Templates server-side con EJS
- **Base de Datos:** MySQL
- **ORM:** Sequelize
- **CSS:** Bootstrap 5

## 6. Integrantes del Grupo

| Apellido, Nombre | Specs Asignadas |
|---|---|
| Caruk, Maria Eugenia | 01 - Autenticación y Gestión de Usuarios / 02 - Gestión de Roles y Perfiles |
| Giménez, Joaquin Elian | 03 - Gestión de Eventos / 04 - Inscripciones |
| Ivasiuta, Magdalena Ramona | 05 - Certificados / 06 - Encuestas y Comentarios |
| Thoux, Ivan Ezequiel | 07 - Informes y Agenda / 08 - Acreditación y Notificaciones |
