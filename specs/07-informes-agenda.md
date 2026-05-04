# Spec 07 - Informes y Agenda

**Integrante:** Thoux, Ivan Ezequiel

## 1. Objetivo y Contexto

Implementar la generación de informes y la agenda del evento. Este módulo proporciona a los organizadores y participantes datos estadísticos sobre los eventos: cantidad de asistentes, estadísticas de encuestas, listado de disertantes, y una agenda detallada del evento.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles), Spec 03 (Eventos), Spec 04 (Inscripciones), Spec 05 (Certificados), Spec 06 (Encuestas).
**Consumido por:** Ninguno directamente, pero consume datos de todos los módulos anteriores.

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Ver informe de asistentes
**Como** organizador de un evento, **quiero** ver un informe con la cantidad de asistentes, **para** conocer el nivel de participación.

**Criterios de Aceptación:**
- CA-01: El informe muestra: total de inscritos, total de acreditados, total en lista de espera, tasa de asistencia (acreditados/inscritos * 100).
- CA-02: Se muestra un gráfico de barras con inscritos vs. acreditados (usando Chart.js o similar en el cliente).
- CA-03: Se puede filtrar por rango de fechas si se seleccionan múltiples eventos.
- CA-04: Solo el organizador del evento puede ver este informe.

### HU-02: Ver agenda del evento
**Como** participante o visitante, **quiero** ver la agenda del evento, **para** conocer el horario y las actividades programadas.

**Criterios de Aceptación:**
- CA-01: La agenda muestra: fecha, horario, actividad, disertante, lugar/sala.
- CA-02: La agenda se visualiza en formato de timeline o tabla cronológica.
- CA-03: Si el evento tiene múltiples días, la agenda se agrupa por día.
- CA-04: La agenda es pública y accesible sin autenticación.
- CA-05: El organizador puede crear y editar las entradas de la agenda desde su panel.

### HU-03: Ver estadísticas de encuestas
**Como** organizador de un evento, **quiero** ver estadísticas detalladas de las encuestas, **para** evaluar la calidad del evento.

**Criterios de Aceptación:**
- CA-01: Se muestran promedios de cada pregunta numérica (calificación general, contenido, disertantes, organización).
- CA-02: Se muestra la distribución de respuestas (cuántos respondieron 1, 2, 3, 4, 5) en gráfico de barras.
- CA-03: Se muestra el porcentaje de recomendación (Sí vs. No).
- CA-04: Se muestra la cantidad total de encuestas respondidas vs. total de acreditados.
- CA-05: Solo el organizador del evento puede ver estas estadísticas.

### HU-04: Ver listado de disertantes
**Como** organizador o visitante, **quiero** ver el listado de disertantes del evento, **para** conocer quiénes participan.

**Criterios de Aceptación:**
- CA-01: El listado muestra: nombre, apellido, título de la presentación, biografía (si está disponible).
- CA-02: El listado es público y accesible sin autenticación.
- CA-03: Se muestra en la página de detalle del evento y en una sección dedicada.
- CA-04: El organizador puede ver datos adicionales: email, teléfono (si fue proporcionado).

### HU-05: Generar informe general del evento
**Como** organizador de un evento, **quiero** generar un informe completo del evento, **para** tener un documento resumen de todo lo ocurrido.

**Criterios de Aceptación:**
- CA-01: El informe incluye: datos generales del evento, cantidad de asistentes, estadísticas de encuestas, listado de disertantes, agenda, comentarios destacados.
- CA-02: El informe se puede descargar en formato PDF.
- CA-03: El informe se genera al cerrar el evento (estado `completed`) o manualmente desde el panel.
- CA-04: Se guarda una copia del informe en el sistema para consulta futura.

### HU-06: Dashboard del organizador
**Como** organizador, **quiero** ver un dashboard con resumen de mis eventos, **para** tener una vista rápida de mi actividad.

**Criterios de Aceptación:**
- CA-01: El dashboard muestra: total de eventos creados, eventos próximos, eventos completados, total de inscripciones en todos mis eventos.
- CA-02: Se muestra un listado de los últimos 5 eventos con su estado.
- CA-03: Se muestra un gráfico de inscripciones por mes (últimos 6 meses).
- CA-04: Solo el organizador ve su propio dashboard.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Informes de asistentes
- Los datos se calculan a partir de la tabla `registrations` con `status` correspondiente.
- `inscritos` = count donde status = 'confirmed' o 'attended'.
- `acreditados` = count donde status = 'attended'.
- `lista de espera` = count en `waitlist_entries` donde status = 'waiting'.

### RF-02: Agenda del evento
- La agenda se compone de `agenda_items` asociados a un evento.
- Cada entrada tiene: título, descripción, fecha/hora de inicio, fecha/hora de fin, speaker, lugar.
- El organizador puede agregar, editar y eliminar entradas de la agenda.

### RF-03: Estadísticas de encuestas
- Los promedios se calculan con `AVG` sobre las respuestas de `surveys`.
- La distribución se calcula con `COUNT` agrupado por rating.
- El porcentaje de recomendación = (count(would_recommend=1) / total_surveys) * 100.

### RF-04: Listado de disertantes
- Se obtiene de la tabla `event_speakers` con JOIN a `users`.
- Incluye `presentation_title` si fue registrado.

### RF-05: Informe general en PDF
- Se genera con `pdfkit` combinando todas las secciones del informe.
- Se almacena en `public/reports/` con nombre basado en el evento.

### RN-01: Solo el organizador puede ver informes de asistentes y estadísticas de encuestas.
### RN-02: La agenda del evento es pública.
### RN-03: El listado de disertantes es público.
### RN-04: Un informe general solo se puede generar para eventos en estado `completed` o `ongoing`.
### RN-05: El dashboard del organizador solo muestra sus propios eventos.

## 4. Restricciones Técnicas Específicas

- **Gráficos:** usar Chart.js (CDN) en el cliente para gráficos de barras.
- **PDF de informe:** usar `pdfkit` para generar el informe completo.
- **Consultas de agregación:** usar `Sequelize.fn('AVG')`, `Sequelize.fn('COUNT')`, `Sequelize.fn('SUM')`.
- **Performance:** las consultas de estadísticas deben ser eficientes; usar índices en `event_id` y `status`.
- **Caché:** si las consultas son pesadas, considerar caché de resultados por 5 minutos.

## 5. Modelo de Datos

### Tabla: `agenda_items`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| title | VARCHAR(200) | NO | Título de la actividad |
| description | TEXT | YES | Descripción |
| start_time | DATETIME | NO | Fecha y hora de inicio |
| end_time | DATETIME | NO | Fecha y hora de fin |
| speaker_id | INT | YES | Foreign Key -> users.id |
| location | VARCHAR(255) | YES | Sala o lugar |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `reports`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| generated_by | INT | NO | Foreign Key -> users.id |
| file_path | VARCHAR(500) | YES | Ruta al archivo PDF |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
Event.hasMany(AgendaItem, { foreignKey: 'event_id', as: 'agendaItems' });
AgendaItem.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });
AgendaItem.belongsTo(User, { foreignKey: 'speaker_id', as: 'speaker' });

Event.hasMany(Report, { foreignKey: 'event_id', as: 'reports' });
Report.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });
Report.belongsTo(User, { foreignKey: 'generated_by', as: 'generatedBy' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo AgendaItem | Definir modelo | `src/models/AgendaItem.js` |
| T02 | Crear modelo Report | Definir modelo | `src/models/Report.js` |
| T03 | Crear migraciones | Migraciones para agenda_items y reports | `migrations/` |
| T04 | Controller Report - Asistentes | Calcular estadísticas de asistentes | `src/controllers/reportController.js` |
| T05 | Controller Report - Encuestas | Calcular estadísticas de encuestas | `src/controllers/reportController.js` |
| T06 | Controller Report - Disertantes | Listado de disertantes | `src/controllers/reportController.js` |
| T07 | Controller Report - Informe general | Generar PDF completo del evento | `src/controllers/reportController.js` |
| T08 | Controller Agenda - CRUD | Crear, editar, eliminar entradas de agenda | `src/controllers/reportController.js` |
| T09 | Controller Report - Dashboard | Resumen de eventos del organizador | `src/controllers/reportController.js` |
| T10 | Servicio de informes | Lógica de generación de PDF de informe | `src/services/reportService.js` |
| T11 | Rutas de informes | Definir todas las rutas | `src/routes/reports.js` |
| T12 | Vistas - Informe de asistentes | Gráficos y datos | `src/views/reports/attendees.ejs` |
| T13 | Vistas - Agenda | Timeline de agenda | `src/views/reports/agenda.ejs` |
| T14 | Vistas - Estadísticas encuestas | Gráficos de encuestas | `src/views/reports/surveys.ejs` |
| T15 | Vistas - Disertantes | Listado público | `src/views/reports/speakers.ejs` |
| T16 | Vistas - Dashboard organizador | Resumen de actividad | `src/views/reports/dashboard.ejs` |
| T17 | Vistas - Gestión de agenda | CRUD de agenda (organizador) | `src/views/events/agenda-manage.ejs` |
| T18 | Integración Chart.js | Incluir Chart.js en vistas de informes | `src/views/reports/` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Informe de asistentes:** Ver informe con datos correctos de inscritos, acreditados y tasa de asistencia.
2. **Gráfico de asistentes:** Verificar que el gráfico de barras muestra datos correctos.
3. **Agenda del evento:** Crear entradas de agenda. Verificar que se muestran en orden cronológico.
4. **Agenda multiday:** Crear entradas para 2 días. Verificar agrupación por día.
5. **Agenda pública:** Ver agenda sin estar autenticado. Verificar que es visible.
6. **Estadísticas de encuestas:** Completar varias encuestas. Verificar promedios y distribución correctos.
7. **Listado de disertantes:** Verificar que muestra nombre, apellido y título de presentación.
8. **Informe general PDF:** Generar informe. Verificar que incluye todas las secciones.
9. **Dashboard organizador:** Crear varios eventos. Verificar resumen correcto.
10. **Permisos:** Intentar ver informe de asistentes como participante. Verificar 403.

### Tests de Performance
- Generar informe con 500+ inscritos. Verificar que la consulta responde en menos de 3 segundos.
- Verificar que las consultas de agregación usan índices correctamente.
