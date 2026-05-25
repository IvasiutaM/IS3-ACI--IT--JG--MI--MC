# Spec 04 - Inscripciones

**Integrante:** Giménez, Joaquin Elian

## 1. Objetivo y Contexto

Implementar el sistema de inscripciones a eventos. Permite a los participantes inscribirse de forma autónoma (desde la web o enlace directo) y al organizador/disertante inscribir manualmente a participantes. Incluye la gestión automática de lista de espera cuando se alcanza el cupo máximo.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles), Spec 03 (Eventos).
**Consumido por:** Spec 05 (Certificados), Spec 06 (Encuestas), Spec 07 (Informes), Spec 08 (Acreditación).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Inscripción autónoma a evento
**Como** participante registrado, **quiero** inscribirme a un evento desde su página de detalle, **para** asistir al evento.

**Criterios de Aceptación:**
- CA-01: El botón "Inscribirme" aparece en el detalle del evento si: está abierto a inscripciones, hay cupo disponible, y la fecha límite no pasó.
- CA-02: Solo usuarios autenticados y verificados pueden inscribirse.
- CA-03: Un usuario NO puede inscribirse más de una vez al mismo evento.
- CA-04: Si el cupo máximo se alcanzó, el usuario se inscribe automáticamente en la lista de espera.
- CA-05: Si la fecha límite de inscripción ya pasó, se muestra mensaje y se deshabilita el botón.
- CA-06: Tras la inscripción, se muestra confirmación y se envía email de confirmación.

### HU-02: Inscripción por enlace directo
**Como** participante, **quiero** inscribirme a un evento haciendo clic en un enlace compartido, **para** facilitar la inscripción rápida.

**Criterios de Aceptación:**
- CA-01: Cada evento genera un enlace de inscripción única: `/events/:id/register?ref=share`.
- CA-02: El enlace lleva directamente al formulario de inscripción (si está autenticado) o al login.
- CA-03: Tras el login, se redirige automáticamente a la inscripción del evento.
- CA-04: El enlace funciona incluso si el usuario no tenía cuenta (lo lleva al registro primero).

### HU-03: Inscripción por el staff (organizador/disertante)
**Como** organizador o disertante de un evento, **quiero** inscribir manualmente a un participante, **para** registrar a personas que no pueden hacerlo por la web.

**Criterios de Aceptación:**
- CA-01: El organizador/disertante accede a una sección "Inscribir participante" en el panel del evento.
- CA-02: Puede buscar al participante por email. Si existe, lo inscribe directamente.
- CA-03: Si el participante no existe, puede crear una cuenta rápida (nombre, apellido, email) y se envía email de registro.
- CA-04: Se registra quién realizó la inscripción manual (staff member).
- CA-05: Se respetan los cupos y la lista de espera igual que la inscripción autónoma.

### HU-04: Lista de espera automática
**Como** participante, **quiero** que al llenarse el cupo de un evento se me inscriba en lista de espera, **para** ser notificado si se libera un lugar.

**Criterios de Aceptación:**
- CA-01: Cuando las inscripciones alcanzan `max_capacity`, las nuevas inscripciones van a la lista de espera.
- CA-02: La lista de espera es FIFO (primero en entrar, primero en salir).
- CA-03: Si un participante inscrito cancela su inscripción, el primero de la lista de espera recibe automáticamente el lugar.
- CA-04: Se envía email al participante promovido desde la lista de espera.
- CA-05: El participante promovido tiene 48 horas para confirmar; si no confirma, pasa al siguiente.

### HU-05: Cancelar inscripción
**Como** participante inscrito, **quiero** cancelar mi inscripción, **para** liberar mi lugar.

**Criterios de Aceptación:**
- CA-01: El participante puede cancelar su inscripción desde su perfil o desde el detalle del evento.
- CA-02: Se puede cancelar hasta 24 horas antes del inicio del evento.
- CA-03: Al cancelar, si hay lista de espera, se promueve automáticamente al siguiente.
- CA-04: Se envía email de confirmación de cancelación.

### HU-06: Ver mis inscripciones
**Como** participante, **quiero** ver el listado de eventos a los que estoy inscrito, **para** gestionar mi participación.

**Criterios de Aceptación:**
- CA-01: El listado muestra: evento, fecha, estado de inscripción (confirmado, lista de espera, acreditado).
- CA-02: Se puede cancelar la inscripción desde el listado.
- CA-03: Se muestran separados: inscritos confirmados, en lista de espera, eventos pasados.

### HU-07: Gestionar inscripciones (organizador)
**Como** organizador de un evento, **quiero** ver y gestionar las inscripciones de mi evento, **para** controlar la asistencia.

**Criterios de Aceptación:**
- CA-01: Se muestra el listado completo de inscritos con nombre, email, fecha de inscripción.
- CA-02: Se muestra el cupo ocupado vs. cupo máximo.
- CA-03: Se muestra la lista de espera con orden y cantidad de personas.
- CA-04: El organizador puede cancelar la inscripción de un participante (justificación obligatoria).
- CA-05: El organizador puede exportar la lista de inscritos a CSV.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Inscripción
- Una inscripción se registra en `registrations` con `status = 'confirmed'` o `'waitlisted'`.
- Se verifica que no exista una inscripción previa del mismo usuario al mismo evento.

### RF-02: Lista de espera
- Cuando `count(confirmed registrations) >= max_capacity`, las nuevas inscripciones van a la lista de espera con `status = 'waitlisted'`.
- La posición en la lista de espera se determina por `created_at` (orden de llegada).

### RF-03: Promoción automática
- Al cancelarse una inscripción confirmada, el primer `waitlisted` se promueve a `confirmed`.
- Se envía email de promoción con enlace de confirmación (48hs para confirmar).
- Si no confirma en 48hs, se pasa al siguiente de la lista.

### RF-04: Cancelación
- El participante puede cancelar hasta 24hs antes del evento.
- El organizador puede cancelar en cualquier momento con justificación.
- Al cancelar, se dispara la promoción de lista de espera si aplica.

### RN-01: Un usuario no puede inscribirse dos veces al mismo evento.
### RN-02: No se puede cancelar inscripción menos de 24hs antes del evento (solo el organizador puede).
### RN-03: La lista de espera respeta estrictamente el orden FIFO.
### RN-04: Si el evento tiene cupo mínimo y al llegar la fecha no se alcanza, el organizador recibe alerta.
### RN-05: Las inscripciones manuales por staff consumen cupo igual que las autónomas.

## 4. Restricciones Técnicas Específicas

- **Transacciones de BD:** las operaciones de inscripción y promoción de lista de espera deben ser atómicas (usar transacciones de Sequelize).
- **Race condition:** al inscribirse, verificar cupo dentro de una transacción con bloqueo para evitar sobresuscripción.
- **Jobs programados:** implementar un job (con `node-cron`) que revise promociones de lista de espera pendientes de confirmación (48hs).
- **Exportar CSV:** usar la librería `json2csv` o generar CSV manualmente.
- **Email de confirmación:** enviar inmediatamente tras inscripción exitosa.

## 5. Modelo de Datos

### Tabla: `registrations`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| status | ENUM('confirmed', 'waitlisted', 'cancelled', 'attended') | NO | DEFAULT 'confirmed' |
| registration_type | ENUM('self', 'staff', 'waitlist_promoted') | NO | Tipo de inscripción |
| registered_by | INT | YES | Foreign Key -> users.id (staff que inscribió, si aplica) |
| promotion_deadline | DATETIME | YES | Fecha límite para confirmar (si viene de lista de espera) |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `waitlist_entries`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| position | INT | NO | Posición en la lista (auto-incremental por evento) |
| status | ENUM('waiting', 'promoted', 'expired', 'declined') | NO | DEFAULT 'waiting' |
| promoted_at | DATETIME | YES | Fecha de promoción |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `registration_cancellations`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| registration_id | INT | NO | Foreign Key -> registrations.id |
| cancelled_by | INT | NO | Foreign Key -> users.id |
| reason | TEXT | YES | Motivo de la cancelación |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
Event.hasMany(Registration, { foreignKey: 'event_id', as: 'registrations' });
User.hasMany(Registration, { foreignKey: 'user_id', as: 'registrations' });
Registration.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });
Registration.belongsTo(User, { foreignKey: 'user_id', as: 'user' });

Event.hasMany(WaitlistEntry, { foreignKey: 'event_id', as: 'waitlist' });
User.hasMany(WaitlistEntry, { foreignKey: 'user_id', as: 'waitlistEntries' });

Registration.hasOne(RegistrationCancellation, { foreignKey: 'registration_id', as: 'cancellation' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo Registration | Definir modelo con validaciones | `src/models/Registration.js` |
| T02 | Crear modelo WaitlistEntry | Modelo para lista de espera | `src/models/WaitlistEntry.js` |
| T03 | Crear modelo RegistrationCancellation | Registro de cancelaciones | `src/models/RegistrationCancellation.js` |
| T04 | Crear migraciones | Migraciones para las 3 tablas | `migrations/` |
| T05 | Controller Registration - Inscripción autónoma | Lógica con verificación de cupo y transacción | `src/controllers/registrationController.js` |
| T06 | Controller Registration - Inscripción por enlace | Redirección desde enlace compartido | `src/controllers/registrationController.js` |
| T07 | Controller Registration - Inscripción por staff | Buscar/crear usuario e inscribir | `src/controllers/registrationController.js` |
| T08 | Controller Registration - Cancelar | Lógica de cancelación + promoción de espera | `src/controllers/registrationController.js` |
| T09 | Controller Registration - Mis inscripciones | Listado de inscripciones del usuario | `src/controllers/registrationController.js` |
| T10 | Controller Registration - Gestión (organizador) | Listado, cancelación, exportar CSV | `src/controllers/registrationController.js` |
| T11 | Servicio de lista de espera | Lógica de promoción automática FIFO | `src/services/waitlistService.js` |
| T12 | Job programado | Cron job para revisar promociones expiradas | `src/utils/scheduler.js` |
| T13 | Rutas de inscripciones | Definir todas las rutas | `src/routes/registrations.js` |
| T14 | Vistas - Formulario de inscripción | Formulario de inscripción | `src/views/registrations/create.ejs` |
| T15 | Vistas - Mis inscripciones | Listado de inscripciones propias | `src/views/registrations/my-registrations.ejs` |
| T16 | Vistas - Gestión de inscripciones (organizador) | Listado, lista de espera, exportar | `src/views/events/registrations.ejs` |
| T17 | Exportar CSV | Generar archivo CSV de inscritos | `src/utils/csvExport.js` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Inscripción exitosa:** Inscribirse a evento con cupo disponible. Verificar status `confirmed` y email.
2. **Doble inscripción:** Intentar inscribirse dos veces al mismo evento. Verificar rechazo.
3. **Cupo lleno:** Inscribirse cuando se alcanza el cupo máximo. Verificar que queda en lista de espera.
4. **Lista de espera FIFO:** Verificar que el orden de espera respeta el orden de llegada.
5. **Cancelación + promoción:** Cancelar inscripción confirmada. Verificar que el primero de la espera se promueve.
6. **Promoción expirada:** Simular que el promovido no confirma en 48hs. Verificar que pasa al siguiente.
7. **Inscripción por staff:** Como organizer, inscribir a otro usuario. Verificar status y `registered_by`.
8. **Enlace directo:** Probar inscripción desde enlace compartido con usuario no logueado.
9. **Cancelación tardía:** Intentar cancelar menos de 24hs antes. Verificar que solo el organizer puede.
10. **Exportar CSV:** Exportar lista de inscritos. Verificar que el archivo es correcto.

### Tests de Concurrencia
- Simular 2 inscripciones simultáneas al último cupo disponible. Verificar que solo una se confirma y la otra va a espera.

### Tests de Seguridad
- Intentar inscribirse a evento que no existe.
- Intentar cancelar inscripción de otro usuario.
- Intentar acceder a gestión de inscripciones sin ser organizer del evento.
### Mejora 
Integrante: Giménez, Joaquin Elian
Módulo: Spec 04 - Inscripciones 
Riesgo OWASP a mitigar: A01:2021 - Broken Access Control (Control de Acceso Roto - Específicamente BOLA/IDOR: Insecure Direct Object References).
Historia de usuario modificada:
HU-01: Inscripción autónoma a evento 
Como participante registrado, quiero inscribirme a un evento desde su página de detalle, para asistir al evento.
Criterios de aceptación originales: CA-01 al CA-06 (Validación de cupos, lista de espera, etc.).
Nuevos CA enriquecidos (Controles OWASP): 
CA-07 (OWASP Access Control): la identidad del usuario que se inscribe debe extraerse exclusivamente del token/sesión del servidor (req.session.userId) y nunca de un parámetro enviado en el cuerpo de la petición (ej. oculto en un formulario HTML) para evitar que un usuario inscriba a otro manipulando el ID (IDOR). 
CA-08 (OWASP Business Logic): la validación de que un usuario "NO puede inscribirse más de una vez" debe ejecutarse dentro de la transacción de base de datos de Sequelize con un bloqueo concurrente, impidiendo vulnerabilidades de condiciones de carrera (Race Conditions).