# Spec 03 - Gestión de Eventos

**Integrante:** Giménez, Joaquin Elian

## 1. Objetivo y Contexto

Implementar la funcionalidad completa de gestión de eventos académicos. Este módulo permite a los organizadores crear, editar, visualizar y eliminar eventos, y al administrador gestionar los tipos de evento. El listado de eventos es público y accesible sin autenticación.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles y Perfiles).
**Consumido por:** Spec 04 (Inscripciones), Spec 05 (Certificados), Spec 06 (Encuestas), Spec 07 (Informes), Spec 08 (Acreditación).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Listado público de eventos
**Como** visitante del sitio, **quiero** ver el listado de eventos, **para** conocer los eventos disponibles y sus detalles.

**Criterios de Aceptación:**
- CA-01: El listado muestra: nombre, tipo, fecha de inicio, fecha de fin, lugar, estado (próximo/en curso/finalizado).
- CA-02: El listado está paginado (20 eventos por página).
- CA-03: Se puede filtrar por tipo de evento, rango de fechas y estado (próximo, en curso, pasado).
- CA-04: Los eventos se ordenan por fecha de inicio (más próximos primero).
- CA-05: No se requiere autenticación para ver el listado.

### HU-02: Detalle de evento
**Como** visitante, **quiero** ver los detalles completos de un evento, **para** decidir si me interesa participar.

**Criterios de Aceptación:**
- CA-01: Se muestra: nombre, descripción, tipo, fechas (inicio, fin, límite de inscripción), lugar, cupo (disponible/máximo), organizador, disertantes.
- CA-02: Si el evento está abierto a inscripciones, se muestra botón "Inscribirme".
- CA-03: Si el evento está lleno, se muestra mensaje "Cupo completo - Lista de espera disponible".
- CA-04: Si la fecha límite de inscripción ya pasó, se muestra mensaje "Inscripciones cerradas".

### HU-03: Crear evento
**Como** organizador, **quiero** crear un nuevo evento, **para** ofrecer actividades académicas.

**Criterios de Aceptación:**
- CA-01: El formulario solicita: nombre, descripción, tipo de evento, fecha de inicio, fecha de fin, fecha límite de inscripción, lugar, cupo mínimo, cupo máximo.
- CA-02: La fecha de inicio debe ser posterior a la fecha actual.
- CA-03: La fecha de fin debe ser igual o posterior a la fecha de inicio.
- CA-04: La fecha límite de inscripción debe ser anterior a la fecha de inicio.
- CA-05: El cupo máximo debe ser mayor que el cupo mínimo.
- CA-06: El cupo mínimo es opcional (puede ser 0).
- CA-07: Al crear el evento, el usuario creador se asigna automáticamente como organizador del evento.

### HU-04: Editar evento
**Como** organizador de un evento, **quiero** editar los datos de mi evento, **para** corregir información o actualizar detalles.

**Criterios de Aceptación:**
- CA-01: Solo el organizador que creó el evento puede editarlo.
- CA-02: Se pueden modificar todos los campos del evento.
- CA-03: No se puede editar un evento que ya comenzó.
- CA-04: Si se reduce el cupo máximo por debajo de los inscritos actuales, se muestra advertencia.

### HU-05: Eliminar evento
**Como** organizador de un evento, **quiero** eliminar un evento, **para** cancelar un evento que ya no se realizará.

**Criterios de Aceptación:**
- CA-01: Solo el organizador que creó el evento puede eliminarlo.
- CA-02: No se puede eliminar un evento que ya comenzó.
- CA-03: Al eliminar, se solicita confirmación.
- CA-04: Se envía email de notificación a todos los inscritos informando la cancelación.
- CA-05: El evento se marca como `deleted = true` (soft delete), no se elimina físicamente.

### HU-06: Gestionar tipos de evento
**Como** administrador de la plataforma, **quiero** crear, editar y eliminar tipos de evento, **para** categorizar correctamente los eventos.

**Criterios de Aceptación:**
- CA-01: El administrador puede crear nuevos tipos con nombre y descripción.
- CA-02: Los tipos existentes por defecto son: Curso, Jornada, Congreso, Charla, Seminario, Taller.
- CA-03: No se puede eliminar un tipo que tiene eventos asociados.
- CA-04: Solo el administrador puede gestionar tipos de evento.

### HU-07: Asignar disertantes al evento
**Como** organizador de un evento, **quiero** asignar disertantes a mi evento, **para** definir quiénes expondrán.

**Criterios de Aceptación:**
- CA-01: El organizador puede buscar usuarios existentes por nombre o email.
- CA-02: Puede asignar múltiples disertantes a un evento.
- CA-03: Al asignar un disertante, el usuario recibe el rol `speaker` si no lo tiene.
- CA-04: Se puede quitar un disertante del evento.
- CA-05: Se muestra el listado de disertantes en la vista de detalle del evento.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Creación de eventos
- Solo usuarios con rol `organizer` pueden crear eventos.
- El creador del evento se registra automáticamente como su organizador en la tabla `event_organizers`.

### RF-02: Tipos de evento
- Los tipos de evento son creados exclusivamente por el administrador.
- Un evento DEBE tener un tipo asociado.

### RF-03: Estados del evento
- `upcoming`: La fecha de inicio es futura.
- `ongoing`: La fecha actual está entre inicio y fin.
- `completed`: La fecha de fin ya pasó.
- `cancelled`: El evento fue cancelado por el organizador.
- El estado se calcula dinámicamente según las fechas, excepto `cancelled`.

### RF-04: Cupos
- `min_capacity`: cupo mínimo para que el evento se realice (puede ser 0).
- `max_capacity`: cupo máximo de participantes.
- Si las inscripciones no alcanzan el cupo mínimo 48hs antes del evento, el organizador recibe una alerta.

### RF-05: Fechas
- `start_date`: fecha y hora de inicio del evento.
- `end_date`: fecha y hora de fin del evento.
- `registration_deadline`: fecha límite para inscribirse.

### RN-01: Un evento no puede tener fecha de inicio anterior a la fecha actual.
### RN-02: Un evento no puede editarse ni eliminarse después de su fecha de inicio.
### RN-03: Solo el administrador puede crear tipos de evento.
### RN-04: Solo el organizador que creó un evento puede editarlo o eliminarlo.
### RN-05: No se puede eliminar un tipo de evento que tenga eventos asociados.
### RN-06: Al eliminar un evento, se notifica a todos los inscritos.

## 4. Restricciones Técnicas Específicas

- **Soft delete:** usar campo `deleted_at` en lugar de eliminar físicamente los eventos.
- **Validación de fechas:** validar en controlador y en vista (HTML5 date inputs).
- **Filtros:** implementar paginación con Sequelize (`limit`, `offset`, `order`).
- **Búsqueda:** permitir búsqueda por nombre con `LIKE`.
- **Cálculo de estado:** función helper que determine el estado según fechas actuales.
- **Relación N:M:** eventos pueden tener múltiples organizadores y múltiples disertantes.

## 5. Modelo de Datos

### Tabla: `event_types`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| name | VARCHAR(100) | NO | Nombre del tipo (Curso, Jornada, etc.) |
| slug | VARCHAR(50) | NO | UNIQUE, identificador (curso, jornada, etc.) |
| description | TEXT | YES | Descripción |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `events`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| name | VARCHAR(200) | NO | Nombre del evento |
| description | TEXT | NO | Descripción completa |
| event_type_id | INT | NO | Foreign Key -> event_types.id |
| organizer_id | INT | NO | Foreign Key -> users.id (creador) |
| start_date | DATETIME | NO | Fecha y hora de inicio |
| end_date | DATETIME | NO | Fecha y hora de fin |
| registration_deadline | DATETIME | NO | Fecha límite de inscripción |
| location | VARCHAR(255) | NO | Lugar del evento |
| min_capacity | INT | NO | DEFAULT 0, cupo mínimo |
| max_capacity | INT | YES | NULL = sin límite |
| status | ENUM('upcoming', 'ongoing', 'completed', 'cancelled') | NO | DEFAULT 'upcoming' |
| is_active | TINYINT(1) | NO | DEFAULT 1 (soft delete) |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `event_organizers`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `event_speakers`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| presentation_title | VARCHAR(255) | YES | Título de la presentación |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
EventType.hasMany(Event, { foreignKey: 'event_type_id', as: 'events' });
Event.belongsTo(EventType, { foreignKey: 'event_type_id', as: 'eventType' });

Event.belongsTo(User, { foreignKey: 'organizer_id', as: 'organizer' });
Event.belongsToMany(User, { through: 'event_organizers', as: 'organizers', foreignKey: 'event_id' });
Event.belongsToMany(User, { through: 'event_speakers', as: 'speakers', foreignKey: 'event_id' });

EventSpeaker.belongsTo(User, { foreignKey: 'user_id', as: 'speaker' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo EventType | Definir modelo | `src/models/EventType.js` |
| T02 | Crear modelo Event | Definir modelo con validaciones | `src/models/Event.js` |
| T03 | Crear modelo EventOrganizer | Tabla intermedia | `src/models/EventOrganizer.js` |
| T04 | Crear modelo EventSpeaker | Tabla intermedia con título | `src/models/EventSpeaker.js` |
| T05 | Crear migraciones | Migraciones para las 4 tablas | `migrations/` |
| T06 | Seed de tipos de evento | Insertar tipos por defecto | `seeders/` |
| T07 | Controller EventType - CRUD | Crear, listar, editar, eliminar tipos | `src/controllers/eventTypeController.js` |
| T08 | Controller Event - Listado público | Listar con filtros y paginación | `src/controllers/eventController.js` |
| T09 | Controller Event - Detalle | Ver detalle de un evento | `src/controllers/eventController.js` |
| T10 | Controller Event - Crear | Crear evento con validaciones | `src/controllers/eventController.js` |
| T11 | Controller Event - Editar | Editar evento propio | `src/controllers/eventController.js` |
| T12 | Controller Event - Eliminar | Soft delete con notificación | `src/controllers/eventController.js` |
| T13 | Controller Event - Asignar disertantes | Agregar/quitar speakers | `src/controllers/eventController.js` |
| T14 | Helper de estado | Función que calcula estado por fechas | `src/utils/helpers.js` |
| T15 | Rutas de eventos | Definir todas las rutas | `src/routes/events.js` |
| T16 | Rutas de tipos | Rutas admin para tipos | `src/routes/eventTypes.js` |
| T17 | Vistas - Listado público | Listado con filtros | `src/views/events/index.ejs` |
| T18 | Vistas - Detalle | Detalle del evento | `src/views/events/show.ejs` |
| T19 | Vistas - Crear/Editar | Formularios | `src/views/events/create.ejs`, `edit.ejs` |
| T20 | Vistas - Tipos (admin) | CRUD de tipos | `src/views/admin/event-types/` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Listado público:** Verificar que se ven todos los eventos activos sin autenticación.
2. **Filtros:** Filtrar por tipo, fecha, estado. Verificar resultados correctos.
3. **Crear evento:** Como organizer, crear evento con datos válidos. Verificar en BD.
4. **Validación de fechas:** Intentar crear evento con fecha de inicio pasada. Verificar rechazo.
5. **Editar evento:** Editar evento propio. Verificar cambios.
6. **Editar evento ajeno:** Intentar editar evento de otro organizer. Verificar 403.
7. **Eliminar evento:** Eliminar evento propio. Verificar soft delete y notificación.
8. **Eliminar evento iniciado:** Intentar eliminar evento que ya comenzó. Verificar rechazo.
9. **Asignar disertante:** Buscar usuario y asignar como speaker. Verificar en detalle.
10. **Gestión de tipos (admin):** Crear, editar, eliminar tipo. Verificar que no se elimina con eventos asociados.
11. **Paginación:** Crear más de 20 eventos. Verificar paginación correcta.

### Tests de Seguridad
- Intentar crear evento sin rol organizer.
- Intentar acceder a rutas de admin de tipos sin ser admin.
- Verificar que los eventos eliminados no aparecen en el listado público.
