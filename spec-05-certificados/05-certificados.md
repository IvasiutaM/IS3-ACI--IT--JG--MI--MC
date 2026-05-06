# Spec 05 - Certificados

**Integrante:** Ivasiuta, Magdalena Ramona

## 1. Objetivo y Contexto

Implementar la generación automática de certificados en formato PDF para los participantes de eventos académicos. El sistema genera 4 tipos de certificados (asistencia, aprobación, disertante/expositor, autor) bajo diferentes condiciones, y permite a los usuarios descargar sus certificados desde su perfil.

**Dependencias:** Spec 01 (Autenticación), Spec 02 (Roles), Spec 03 (Eventos), Spec 04 (Inscripciones), Spec 06 (Encuestas y Comentarios).
**Consumido por:** Spec 07 (Informes - listado de certificados emitidos).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Certificado de asistencia
**Como** participante de un evento, **quiero** obtener un certificado de asistencia, **para** acreditar que asistí al evento.

**Criterios de Aceptación:**
- CA-01: El certificado se genera automáticamente cuando el participante completa la encuesta de satisfacción post-evento.
- CA-02: El certificado incluye: nombre del participante, nombre del evento, fecha del evento, tipo de evento, duración estimada.
- CA-03: El certificado incluye el nombre del organizador y la firma digital del evento.
- CA-04: Se genera en formato PDF descargable.
- CA-05: Se notifica al usuario por email que su certificado está disponible.

### HU-02: Certificado de aprobación
**Como** participante de un evento, **quiero** obtener un certificado de aprobación, **para** acreditar que aprobé el evento.

**Criterios de Aceptación:**
- CA-01: El certificado se genera cuando el disertante evalúa las encuestas del participante y las considera conformes.
- CA-02: El disertante tiene una interfaz donde revisa las respuestas de las encuestas de cada participante.
- CA-03: El disertante puede aprobar individualmente a cada participante o en lote.
- CA-04: El certificado incluye los mismos datos que el de asistencia, más la mención "APROBADO".
- CA-05: Se genera en formato PDF descargable.
- CA-06: Se notifica al usuario por email que su certificado de aprobación está disponible.

### HU-03: Certificado de disertante/expositor
**Como** disertante de un evento, **quiero** obtener un certificado que acredite mi participación como expositor, **para** incluir en mi curriculum.

**Criterios de Aceptación:**
- CA-01: El certificado se genera automáticamente cuando el evento se marca como `completed` (finalizado).
- CA-02: El certificado incluye: nombre del disertante, nombre del evento, fecha del evento, título de la presentación (si fue registrado).
- CA-03: El certificado menciona explícitamente la calidad de "DISERTANTE" o "EXPOSITOR".
- CA-04: Se genera en formato PDF descargable.

### HU-04: Certificado de autor
**Como** participante que presentó un trabajo en un evento, **quiero** obtener un certificado como autor, **para** acreditar mi contribución.

**Criterios de Aceptación:**
- CA-01: El certificado se genera automáticamente cuando el evento se marca como `completed` y el participante tiene el rol de autor registrado.
- CA-02: El organizador puede registrar participantes como "autores" desde el panel del evento.
- CA-03: El certificado incluye: nombre del autor, nombre del evento, fecha, título del trabajo presentado.
- CA-04: Se genera en formato PDF descargable.

### HU-05: Descargar certificados desde mi perfil
**Como** usuario, **quiero** ver y descargar todos mis certificados desde mi perfil, **para** acceder a ellos cuando los necesite.

**Criterios de Aceptación:**
- CA-01: El perfil del usuario muestra una sección "Mis Certificados" con todos los certificados generados.
- CA-02: Cada certificado muestra: tipo, evento, fecha de emisión.
- CA-03: Se puede descargar el PDF haciendo clic en un botón.
- CA-04: Los certificados se ordenan por fecha de emisión (más recientes primero).

### HU-06: Verificar autenticidad de certificado
**Como** tercero, **quiero** verificar que un certificado es auténtico, **para** confirmar su validez.

**Criterios de Aceptación:**
- CA-01: Cada certificado PDF incluye un código de verificación único.
- CA-02: Existe una página pública `/certificates/verify/:code` donde se puede ingresar el código.
- CA-03: Si el código es válido, se muestran los datos del certificado (nombre, evento, tipo, fecha).
- CA-04: Si el código es inválido, se muestra "Certificado no encontrado".

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Generación de certificados
- Los certificados se generan en formato PDF usando `pdfkit`.
- Cada certificado tiene un código de verificación único (`verification_code`).
- El código se genera con `crypto.randomBytes(16).toString('hex')`.

### RF-02: Condiciones de generación por tipo
- **Asistencia:** se genera automáticamente al completar la encuesta post-evento (`surveys` con `status = 'completed'`).
- **Aprobación:** se genera cuando el disertante marca al participante como `approved` en las respuestas de la encuesta.
- **Disertante:** se genera automáticamente cuando el evento cambia a estado `completed` para todos los speakers.
- **Autor:** se genera automáticamente cuando el evento cambia a estado `completed` para los usuarios marcados como autores.

### RF-03: Diseño del PDF
- Formato A4 vertical.
- Encabezado con logo del sistema (placeholder).
- Título: "Certificado de [Asistencia/Aprobación/Disertante/Autor]".
- Cuerpo con datos del participante, evento, fecha y tipo.
- Pie con código de verificación y firma digital (texto).

### RF-04: Almacenamiento
- Los certificados se guardan en la tabla `certificates` con referencia al archivo PDF generado.
- Los archivos PDF se almacenan en `public/certificates/` con nombre basado en el `verification_code`.

### RN-01: Un participante solo puede obtener un certificado de asistencia por evento.
### RN-02: Un participante solo puede obtener un certificado de aprobación por evento, y solo si el disertante lo aprueba.
### RN-03: Un disertante recibe un certificado de disertante por cada evento en el que participó como speaker.
### RN-04: No se pueden generar certificados para eventos que no están en estado `completed` (excepto asistencia y aprobación que se generan post-evento).
### RN-05: El código de verificación es único e irrepetible.

## 4. Restricciones Técnicas Específicas

- **PDF:** usar `pdfkit` para generación de PDFs.
- **Almacenamiento:** archivos en `public/certificates/` con nombres `cert_{verification_code}.pdf`.
- **Generación automática:** escuchar el evento de cambio de estado del evento (`completed`) para generar certificados de disertante y autor.
- **Trigger de encuesta:** al completar la encuesta, generar certificado de asistencia.
- **Código de verificación:** `crypto.randomBytes(16).toString('hex')` -> 32 caracteres hexadecimales.

## 5. Modelo de Datos

### Tabla: `certificates`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| user_id | INT | NO | Foreign Key -> users.id |
| event_id | INT | NO | Foreign Key -> events.id |
| certificate_type | ENUM('attendance', 'approval', 'speaker', 'author') | NO | Tipo de certificado |
| verification_code | CHAR(32) | NO | UNIQUE, código de verificación |
| file_path | VARCHAR(500) | YES | Ruta al archivo PDF |
| metadata | JSON | YES | Datos adicionales (título presentación, trabajo, etc.) |
| issued_at | DATETIME | NO | Fecha de emisión |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `certificate_authors`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| user_id | INT | NO | Foreign Key -> users.id |
| work_title | VARCHAR(255) | YES | Título del trabajo presentado |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `approval_records`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| event_id | INT | NO | Foreign Key -> events.id |
| participant_id | INT | NO | Foreign Key -> users.id |
| speaker_id | INT | NO | Foreign Key -> users.id (disertante que aprobó) |
| approved | TINYINT(1) | NO | DEFAULT 0 |
| notes | TEXT | YES | Notas del disertante |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
User.hasMany(Certificate, { foreignKey: 'user_id', as: 'certificates' });
Event.hasMany(Certificate, { foreignKey: 'event_id', as: 'certificates' });
Certificate.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
Certificate.belongsTo(Event, { foreignKey: 'event_id', as: 'event' });

Event.belongsToMany(User, { through: 'certificate_authors', as: 'authors', foreignKey: 'event_id' });
User.belongsToMany(Event, { through: 'certificate_authors', as: 'authoredEvents', foreignKey: 'user_id' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo Certificate | Definir modelo con validaciones | `src/models/Certificate.js` |
| T02 | Crear modelo CertificateAuthor | Tabla intermedia para autores | `src/models/CertificateAuthor.js` |
| T03 | Crear modelo ApprovalRecord | Registro de aprobaciones por disertante | `src/models/ApprovalRecord.js` |
| T04 | Crear migraciones | Migraciones para las 3 tablas | `migrations/` |
| T05 | Servicio PDF | Crear servicio de generación de PDFs con pdfkit | `src/services/pdfService.js` |
| T06 | Controller Certificate - Generar asistencia | Generar al completar encuesta | `src/controllers/certificateController.js` |
| T07 | Controller Certificate - Generar aprobación | Generar cuando disertante aprueba | `src/controllers/certificateController.js` |
| T08 | Controller Certificate - Generar disertante | Generar al cerrar evento | `src/controllers/certificateController.js` |
| T09 | Controller Certificate - Generar autor | Generar al cerrar evento | `src/controllers/certificateController.js` |
| T10 | Controller Certificate - Mis certificados | Listado de certificados del usuario | `src/controllers/certificateController.js` |
| T11 | Controller Certificate - Descargar | Descargar PDF | `src/controllers/certificateController.js` |
| T12 | Controller Certificate - Verificar | Página pública de verificación | `src/controllers/certificateController.js` |
| T13 | Rutas de certificados | Definir todas las rutas | `src/routes/certificates.js` |
| T14 | Vistas - Mis certificados | Listado y descarga | `src/views/certificates/my-certificates.ejs` |
| T15 | Vistas - Verificación | Página pública de verificación | `src/views/certificates/verify.ejs` |
| T16 | Vistas - Aprobaciones (disertante) | Interfaz para aprobar participantes | `src/views/surveys/approvals.ejs` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Certificado de asistencia:** Completar encuesta post-evento. Verificar que se genera certificado y llega email.
2. **Certificado de aprobación:** Como disertante, aprobar a un participante. Verificar que se genera certificado.
3. **Certificado de disertante:** Marcar evento como `completed`. Verificar que todos los speakers reciben certificado.
4. **Certificado de autor:** Registrar autor en evento, marcar como `completed`. Verificar generación.
5. **Descarga:** Descargar PDF desde perfil. Verificar que se abre correctamente.
6. **Verificación:** Ingresar código de verificación en página pública. Verificar datos correctos.
7. **Código inválido:** Ingresar código inexistente. Verificar mensaje "no encontrado".
8. **Duplicado:** Intentar generar certificado de asistencia dos veces. Verificar que no se duplica.
9. **PDF contenido:** Abrir PDF y verificar que contiene todos los campos requeridos.
10. **Orden de certificados:** Verificar que en el perfil se ordenan por fecha (más recientes primero).

### Tests de Seguridad
- Intentar descargar certificado de otro usuario.
- Intentar verificar con código manipulado.
- Verificar que el archivo PDF no permite modificación del contenido.
