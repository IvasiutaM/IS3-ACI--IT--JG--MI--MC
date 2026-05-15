# Spec 06 - Encuestas y Comentarios

**Integrante:** Ivasiuta, Magdalena Ramona

## 1. Objetivo y Contexto

Implementar el sistema de encuestas de satisfacción post-evento y comentarios públicos por evento. La encuesta es un formulario fijo que los participantes completan después del evento. Los comentarios permiten a los usuarios opinar públicamente sobre el evento.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles), Spec 03 (Eventos), Spec 04 (Inscripciones).
**Consumido por:** Spec 05 (Certificados - la encuesta genera certificado de asistencia), Spec 07 (Informes - estadísticas de encuestas).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Completar encuesta post-evento
**Como** participante que asistió a un evento, **quiero** completar una encuesta de satisfacción, **para** dar mi opinión y obtener mi certificado de asistencia.

**Criterios de Aceptación:**
- CA-01: La encuesta está disponible solo después de que el evento finalizó (estado `completed`).
- CA-02: La encuesta tiene un formulario fijo con las siguientes preguntas:
  - Calificación general del evento (1-5 estrellas).
  - Calidad del contenido (1-5).
  - Calidad de los disertantes (1-5).
  - Organización del evento (1-5).
  - ¿Recomendaría este evento? (Sí/No).
  - Comentarios adicionales (texto libre, opcional).
- CA-03: Solo los participantes inscritos y acreditados pueden completar la encuesta.
- CA-04: Cada participante puede completar la encuesta una sola vez por evento.
- CA-05: Al enviar la encuesta, se genera automáticamente el certificado de asistencia.
- CA-06: Se muestra mensaje de éxito con enlace al certificado generado.

### HU-02: Ver resultados de encuestas (organizador)
**Como** organizador de un evento, **quiero** ver las respuestas de las encuestas, **para** evaluar la satisfacción de los participantes.

**Criterios de Aceptación:**
- CA-01: El organizador accede a una sección "Resultados de Encuestas" en el panel del evento.
- CA-02: Se muestran promedios por cada pregunta numérica (calificación general, contenido, disertantes, organización).
- CA-03: Se muestra el porcentaje de recomendación (Sí/No).
- CA-04: Se muestran los comentarios adicionales de forma anónima.
- CA-05: Se muestra la cantidad de encuestas respondidas vs. total de participantes.

### HU-03: Evaluar encuestas para aprobación (disertante)
**Como** disertante de un evento, **quiero** revisar las respuestas de las encuestas de cada participante, **para** decidir quién recibe certificado de aprobación.

**Criterios de Aceptación:**
- CA-01: El disertante accede a una sección "Evaluación de Aprobaciones" en el panel del evento.
- CA-02: Puede ver las respuestas de cada participante de forma individual.
- CA-03: Puede aprobar o rechazar a cada participante individualmente.
- CA-04: Puede aprobar a todos en lote.
- CA-05: Al aprobar, se genera automáticamente el certificado de aprobación.
- CA-06: Puede agregar notas a la aprobación/rechazo.

### HU-04: Publicar comentario en evento
**Como** usuario autenticado, **quiero** dejar un comentario público en la página del evento, **para** compartir mi opinión.

**Criterios de Aceptación:**
- CA-01: El formulario de comentario aparece en la página de detalle del evento.
- CA-02: Solo usuarios autenticados y verificados pueden comentar.
- CA-03: El comentario tiene un campo de texto (máximo 500 caracteres) y una calificación opcional (1-5).
- CA-04: El comentario se publica inmediatamente y es visible para todos.
- CA-05: Se muestra el nombre del autor y la fecha del comentario.

### HU-05: Ver comentarios de un evento
**Como** visitante, **quiero** ver los comentarios de un evento, **para** conocer las opiniones de otros usuarios.

**Criterios de Aceptación:**
- CA-01: Los comentarios se muestran en la página de detalle del evento.
- CA-02: Se ordenan por fecha (más recientes primero).
- CA-03: Se muestran paginados (10 por página).
- CA-04: Cada comentario muestra: autor, fecha, calificación (si la tiene), texto.
- CA-05: No se requiere autenticación para ver comentarios.

### HU-06: Editar y eliminar comentarios propios
**Como** autor de un comentario, **quiero** editar o eliminar mi comentario, **para** corregir o quitar mi opinión.

**Criterios de Aceptación:**
- CA-01: El autor puede editar su comentario dentro de las 24hs de publicado.
- CA-02: El autor puede eliminar su comentario en cualquier momento.
- CA-03: Se muestra indicador "editado" si el comentario fue modificado.
- CA-04: No se pueden eliminar comentarios de otros usuarios.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Encuesta fija
- La encuesta tiene un formato fijo predefinido. No es configurable por el organizador.
- Las preguntas numéricas usan escala del 1 al 5.
- La encuesta se habilita automáticamente cuando el evento pasa a estado `completed`.

### RF-02: Generación de certificado
- Al completar la encuesta, se dispara automáticamente la generación del certificado de asistencia.
- Si el participante ya tenía un certificado de asistencia, no se genera otro.

### RF-03: Aprobación por disertante
- El disertante revisa las respuestas de cada participante.
- Al aprobar, se genera el certificado de aprobación.
- El disertante puede agregar notas a su decisión.

### RF-04: Comentarios
- Los comentarios son públicos y visibles sin autenticación.
- Solo usuarios autenticados pueden crear, editar o eliminar comentarios.
- Un comentario puede tener una calificación opcional (1-5).

### RN-01: Solo participantes inscritos y acreditados pueden completar la encuesta.
### RN-02: Cada participante completa la encuesta una sola vez por evento.
### RN-03: La encuesta solo está disponible después de que el evento finalizó.
### RN-04: Los comentarios tienen un máximo de 500 caracteres.
### RN-05: Los comentarios se pueden editar solo dentro de las 24hs posteriores a su publicación.
### RN-06: El organizador puede eliminar cualquier comentario de su evento.

## 4. Restricciones Técnicas Específicas

- **Encuesta:** formulario fijo con validación en cliente y servidor.
- **Promedios:** calcular con funciones de agregación de Sequelize (`AVG`, `COUNT`).
- **Comentarios:** usar paginación con `limit` y `offset`.
- **Sanitización:** sanitizar el texto de los comentarios para prevenir XSS (`sanitize-html`).
- **Trigger:** al enviar encuesta, llamar al servicio de certificados para generar el de asistencia.

## 5. Modelo de Datos

### Tabla: `surveys`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| overall_rating | INT | NO | 1-5, calificación general |
| content_rating | INT | NO | 1-5, calidad del contenido |
| speaker_rating | INT | NO | 1-5, calidad de los disertantes |
| organization_rating | INT | NO | 1-5, organización |
| would_recommend | TINYINT(1) | NO | 1=Sí, 0=No |
| additional_comments | TEXT | YES | Comentarios opcionales |
| status | ENUM('pending', 'completed', 'approved', 'rejected') | NO | DEFAULT 'completed' |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `comments`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| content | VARCHAR(500) | NO | Texto del comentario |
| rating | INT | YES | 1-5, calificación opcional |
| is_edited | TINYINT(1) | NO | DEFAULT 0 |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `approval_records`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| participant_id | INT | NO | Foreign Key -> users.id |
| speaker_id | INT | NO | Foreign Key -> users.id (disertante que evaluó) |
| survey_id | INT | YES | Foreign Key -> surveys.id |
| approved | TINYINT(1) | NO | DEFAULT 0 |
| notes | TEXT | YES | Notas del disertante |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
Event.hasMany(Survey, { foreignKey: 'event_id', as: 'surveys' });
User.hasMany(Survey, { foreignKey: 'user_id', as: 'surveys' });
Survey.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });
Survey.belongsTo(User, { foreignKey: 'user_id', as: 'user' });

Event.hasMany(Comment, { foreignKey: 'event_id', as: 'comments' });
User.hasMany(Comment, { foreignKey: 'user_id', as: 'comments' });
Comment.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });
Comment.belongsTo(User, { foreignKey: 'user_id', as: 'user' });

Survey.hasOne(ApprovalRecord, { foreignKey: 'survey_id', as: 'approval' });
ApprovalRecord.belongsTo(Survey, { foreignKey: 'survey_id', as: 'survey' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo Survey | Definir modelo con validaciones | `src/models/Survey.js` |
| T02 | Crear modelo Comment | Definir modelo con sanitización | `src/models/Comment.js` |
| T03 | Crear migraciones | Migraciones para surveys y comments | `migrations/` |
| T04 | Controller Survey - Completar | Lógica de envío de encuesta | `src/controllers/surveyController.js` |
| T05 | Controller Survey - Resultados | Promedios y estadísticas (organizador) | `src/controllers/surveyController.js` |
| T06 | Controller Survey - Evaluación | Interfaz de aprobación (disertante) | `src/controllers/surveyController.js` |
| T07 | Controller Comment - Crear | Crear comentario | `src/controllers/commentController.js` |
| T08 | Controller Comment - Listar | Listar comentarios con paginación | `src/controllers/commentController.js` |
| T09 | Controller Comment - Editar | Editar comentario propio (24hs) | `src/controllers/commentController.js` |
| T10 | Controller Comment - Eliminar | Eliminar comentario propio o como organizador | `src/controllers/commentController.js` |
| T11 | Integración con certificados | Llamar a servicio de certificados al enviar encuesta | `src/controllers/surveyController.js` |
| T12 | Rutas de encuestas | Definir rutas | `src/routes/surveys.js` |
| T13 | Rutas de comentarios | Definir rutas | `src/routes/comments.js` |
| T14 | Vistas - Encuesta | Formulario de encuesta | `src/views/surveys/fill.ejs` |
| T15 | Vistas - Resultados (organizador) | Promedios y comentarios | `src/views/surveys/results.ejs` |
| T16 | Vistas - Evaluación (disertante) | Aprobar/rechazar participantes | `src/views/surveys/evaluation.ejs` |
| T17 | Partial - Comentarios | Sección de comentarios en detalle del evento | `src/views/partials/comments.ejs` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Completar encuesta:** Como participante acreditado, completar encuesta. Verificar que se guarda y se genera certificado.
2. **Duplicado:** Intentar completar encuesta dos veces. Verificar que se rechaza.
3. **Encuesta sin acreditar:** Intentar completar encuesta sin estar acreditado. Verificar rechazo.
4. **Resultados (organizador):** Ver promedios y estadísticas. Verificar cálculos correctos.
5. **Aprobación (disertante):** Aprobar a un participante. Verificar que se genera certificado de aprobación.
6. **Aprobación en lote:** Aprobar a todos los participantes de una vez. Verificar certificados.
7. **Crear comentario:** Como usuario autenticado, crear comentario. Verificar que aparece.
8. **Comentario sin auth:** Intentar comentar sin estar logueado. Verificar redirección a login.
9. **Editar comentario:** Editar dentro de 24hs. Verificar indicador "editado".
10. **Editar comentario tarde:** Intentar editar después de 24hs. Verificar rechazo.
11. **Eliminar comentario propio:** Verificar que desaparece.
12. **Eliminar comentario ajeno:** Intentar eliminar comentario de otro. Verificar 403.
13. **XSS:** Intentar inyectar HTML/JS en comentario. Verificar que se sanitiza.
14. **Paginación de comentarios:** Crear más de 10 comentarios. Verificar paginación.

### Tests de Seguridad
- Verificar sanitización de comentarios.
- Verificar que solo usuarios autenticados pueden crear/editar/eliminar.
- Verificar que el organizador puede eliminar cualquier comentario de su evento.
