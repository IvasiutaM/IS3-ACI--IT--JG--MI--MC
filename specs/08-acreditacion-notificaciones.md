# Spec 08 - Acreditación y Notificaciones

**Integrante:** Thoux, Ivan Ezequiel

## 1. Objetivo y Contexto

Implementar el sistema de acreditación mediante código QR y las notificaciones por email del sistema. La acreditación permite verificar la asistencia de los participantes al momento del ingreso al evento. Las notificaciones envían emails de confirmación de inscripción y recordatorio de evento.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles), Spec 03 (Eventos), Spec 04 (Inscripciones).
**Consumido por:** Spec 05 (Certificados - acreditación genera certificado de asistencia), Spec 06 (Encuestas - encuesta disponible solo para acreditados).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Generar QR de acreditación
**Como** participante inscrito a un evento, **quiero** tener un código QR en mi perfil, **para** presentarlo al llegar al evento y acreditar mi asistencia.

**Criterios de Aceptación:**
- CA-01: El QR se genera automáticamente al confirmar la inscripción a un evento.
- CA-02: El QR contiene un código único que identifica al usuario y al evento.
- CA-03: El QR se muestra en el perfil del usuario, en la sección "Mis Eventos".
- CA-04: El QR se puede descargar como imagen PNG.
- CA-05: El QR también se incluye en el email de confirmación de inscripción.

### HU-02: Acreditar participante (escaneo de QR)
**Como** organizador o staff del evento, **quiero** escanear el QR del participante, **para** registrar su asistencia al evento.

**Criterios de Aceptación:**
- CA-01: El organizador accede a una página de acreditación: `/events/:id/accredit`.
- CA-02: La página muestra la cámara del dispositivo para escanear el QR.
- CA-03: Al escanear un QR válido, se verifica que: el usuario está inscrito, no fue acreditado previamente, el evento está en curso.
- CA-04: Si la acreditación es exitosa, se muestra el nombre del participante y se registra la asistencia.
- CA-05: Si el QR es inválido o ya fue usado, se muestra mensaje de error.
- CA-06: Se lleva un registro de todos los acreditados con fecha y hora.
Nuevos CA enriquecidos (Controles OWASP): 
- CA-07 (OWASP Authorization): el endpoint /events/:id/accredit debe validar estrictamente a nivel de servidor que el usuario que ejecuta la petición HTTP tiene el rol de organizer Y que efectivamente es dueño de ese event_id, impidiendo que un participante manipule la petición para acreditarse a sí mismo. 
- CA-08 (OWASP Data Integrity): el string generado en el QR event:{event_id}:user:{user_id}:reg:{registration_id} debe ser tratado como "no confiable" por el backend. Al escanearlo, el servidor debe cruzar esos tres IDs contra la base de datos para confirmar que la relación es legítima y no fue forjada (falsificada) por un tercero.


### HU-03: Acreditar manualmente
**Como** organizador del evento, **quiero** acreditar manualmente a un participante, **para** los casos donde el QR no funciona o no fue generado.

**Criterios de Aceptación:**
- CA-01: El organizador puede buscar al participante por nombre o email.
- CA-02: Al seleccionar al participante, se registra la acreditación.
- CA-03: Solo se puede acreditar a participantes con inscripción confirmada.
- CA-04: Se registra quién realizó la acreditación manual.

### HU-04: Email de confirmación de inscripción
**Como** participante que se inscribió a un evento, **quiero** recibir un email de confirmación, **para** tener constancia de mi inscripción.

**Criterios de Aceptación:**
- CA-01: El email se envía inmediatamente después de la inscripción exitosa.
- CA-02: El email incluye: nombre del evento, fecha, lugar, enlace al detalle del evento, y el QR de acreditación como imagen adjunta o inline.
- CA-03: Si la inscripción fue a lista de espera, el email indica esa condición.
- CA-04: El email tiene un asunto claro: "Confirmación de inscripción - [Nombre del Evento]".

### HU-05: Email de recordatorio de evento
**Como** participante inscrito, **quiero** recibir un recordatorio antes del evento, **para** no olvidarlo.

**Criterios de Aceptación:**
- CA-01: El email se envía automáticamente 48 horas antes del inicio del evento.
- CA-02: El email incluye: nombre del evento, fecha, hora, lugar, enlace al detalle, y QR de acreditación.
- CA-03: El recordatorio se envía solo a participantes con inscripción confirmada (no en lista de espera).
- CA-04: El asunto del email es: "Recordatorio - [Nombre del Evento] comienza en 48 horas".

### HU-06: Ver listado de acreditados
**Como** organizador del evento, **quiero** ver el listado de participantes acreditados, **para** controlar la asistencia en tiempo real.

**Criterios de Aceptación:**
- CA-01: Se muestra el listado con: nombre, apellido, email, fecha y hora de acreditación.
- CA-02: Se muestra el total de acreditados vs. total de inscritos confirmados.
- CA-03: Se puede buscar por nombre o email.
- CA-04: Se puede exportar a CSV.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Generación de QR
- Cada inscripción confirmada genera un código QR único.
- El código QR contiene un string con formato: `event:{event_id}:user:{user_id}:reg:{registration_id}`.
- El QR se genera usando la librería `qrcode`.
- La imagen del QR se almacena en `public/qrs/` con nombre `qr_{registration_id}.png`.

### RF-02: Proceso de acreditación
- Al acreditar a un participante, se actualiza el `status` de la `registration` a `'attended'`.
- Se registra en la tabla `accreditations` con fecha, hora y quién acreditó.
- Un participante solo puede ser acreditado una vez por evento.
- La acreditación genera automáticamente el certificado de asistencia.

### RF-03: Notificaciones por email
- Los emails se envían usando Nodemailer.
- Se usan templates HTML para los emails.
- Los emails se envían de forma asíncrona (no bloquear la respuesta HTTP).

### RF-04: Job de recordatorios
- Un job programado con `node-cron` revisa cada hora los eventos que comienzan en 48 horas.
- Envía email de recordatorio a todos los inscritos confirmados.
- Se registra en `notification_logs` cada email enviado para evitar duplicados.

### RN-01: Un participante solo puede ser acreditado una vez por evento.
### RN-02: Solo se puede acreditar a participantes con inscripción confirmada.
### RN-03: La acreditación solo puede realizarse el día del evento o después (start_date en adelante).
### RN-04: El email de recordatorio se envía exactamente 48 horas antes del inicio.
### RN-05: No se envían recordatorios a participantes en lista de espera.
### RN-06: Al acreditar, se genera automáticamente el certificado de asistencia.

## 4. Restricciones Técnicas Específicas

- **QR:** usar la librería `qrcode` para generar imágenes PNG.
- **Escaneo de QR:** en el lado del cliente, usar `html5-qrcode` (librería JS) para acceder a la cámara y escanear.
- **Email:** usar Nodemailer con templates EJS para los emails.
- **Jobs:** usar `node-cron` para programar el envío de recordatorios.
- **Asincronía:** los envíos de email no deben bloquear la respuesta HTTP (usar `setImmediate` o colas simples).
- **Log de emails:** registrar cada envío en `notification_logs` para evitar duplicados y para auditoría.

## 5. Modelo de Datos

### Tabla: `accreditations`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| registration_id | INT | NO | Foreign Key -> registrations.id |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| accredited_by | INT | YES | Foreign Key -> users.id (staff que acreditó) |
| method | ENUM('qr_scan', 'manual') | NO | Método de acreditación |
| accredited_at | DATETIME | NO | Fecha y hora de acreditación |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `notification_logs`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| user_id | INT | NO | Foreign Key -> users.id |
| event_id | INT | NO | Foreign Key -> events.id |
| notification_type | ENUM('registration_confirmation', 'event_reminder', 'waitlist_promotion', 'event_cancelled', 'certificate_ready') | NO | Tipo de notificación |
| sent_at | DATETIME | NO | Fecha de envío |
| status | ENUM('sent', 'failed') | NO | Estado del envío |
| error_message | TEXT | YES | Error si el envío falló |

### Relaciones Sequelize
```javascript
Registration.hasOne(Accreditation, { foreignKey: 'registration_id', as: 'accreditation' });
Accreditation.belongsTo(Registration, { foreignKey: 'registration_id', as: 'registration' });
Accreditation.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
Accreditation.belongsTo(User, { foreignKey: 'accredited_by', as: 'accreditedBy' });

User.hasMany(NotificationLog, { foreignKey: 'user_id', as: 'notificationLogs' });
Event.hasMany(NotificationLog, { foreignKey: 'event_id', as: 'notificationLogs' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo Accreditation | Definir modelo | `src/models/Accreditation.js` |
| T02 | Crear modelo NotificationLog | Definir modelo | `src/models/NotificationLog.js` |
| T03 | Crear migraciones | Migraciones para las 2 tablas | `migrations/` |
| T04 | Servicio QR | Generar imágenes QR con librería qrcode | `src/services/qrService.js` |
| T05 | Servicio Email - Confirmación | Template y envío de confirmación | `src/services/emailService.js` |
| T06 | Servicio Email - Recordatorio | Template y envío de recordatorio | `src/services/emailService.js` |
| T07 | Servicio Email - Otros templates | Waitlist, cancelación, certificado | `src/services/emailService.js` |
| T08 | Controller Accreditation - QR | Generar y mostrar QR del participante | `src/controllers/accreditationController.js` |
| T09 | Controller Accreditation - Escanear | Endpoint para escanear QR y acreditar | `src/controllers/accreditationController.js` |
| T10 | Controller Accreditation - Manual | Acreditación manual por búsqueda | `src/controllers/accreditationController.js` |
| T11 | Controller Accreditation - Listado | Listado de acreditados | `src/controllers/accreditationController.js` |
| T12 | Job de recordatorios | Cron job para enviar recordatorios 48hs antes | `src/utils/scheduler.js` |
| T13 | Integración con inscripción | Enviar email de confirmación al inscribirse | `src/controllers/registrationController.js` |
| T14 | Integración con certificados | Generar certificado al acreditar | `src/controllers/accreditationController.js` |
| T15 | Rutas de acreditación | Definir todas las rutas | `src/routes/accreditation.js` |
| T16 | Vistas - QR del participante | Mostrar y descargar QR | `src/views/accreditation/my-qr.ejs` |
| T17 | Vistas - Escáner de QR | Página con cámara para escanear | `src/views/accreditation/scanner.ejs` |
| T18 | Vistas - Listado de acreditados | Tabla con buscador y exportar | `src/views/accreditation/list.ejs` |
| T19 | Templates de email | Crear templates HTML para emails | `src/views/emails/` |
| T20 | Integración html5-qrcode | Incluir librería en vista de escáner | `src/views/accreditation/scanner.ejs` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Generación de QR:** Inscribirse a evento. Verificar que se genera QR en perfil y en email.
2. **Descargar QR:** Descargar imagen PNG. Verificar que se abre correctamente.
3. **Escaneo exitoso:** Escanear QR con la cámara. Verificar que se registra acreditación.
4. **Doble acreditación:** Intentar escanear el mismo QR dos veces. Verificar que se rechaza la segunda.
5. **QR inválido:** Presentar un QR de otro evento. Verificar que se rechaza.
6. **Acreditación manual:** Buscar participante y acreditar manualmente. Verificar registro.
7. **Acreditación fuera de fecha:** Intentar acreditar antes del start_date. Verificar rechazo.
8. **Email de confirmación:** Inscribirse. Verificar que llega el email con QR.
9. **Email de recordatorio:** Modificar fecha de evento a 48hs en el futuro. Ejecutar job manualmente. Verificar email.
10. **Listado de acreditados:** Ver listado después de acreditar varios. Verificar conteo correcto.
11. **Exportar CSV:** Exportar listado de acreditados. Verificar archivo.
12. **Certificado automático:** Acreditar participante. Verificar que se genera certificado de asistencia.

### Tests de Seguridad
- Intentar acreditar sin ser organizer del evento.
- Intentar acceder al escáner sin autenticación.
- Verificar que el QR no es reutilizable.

### Tests de Integración
- Flujo completo: inscripción -> email confirmación -> acreditación QR -> certificado -> encuesta -> certificado asistencia.
