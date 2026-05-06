# Spec 02 - Gestión de Roles y Perfiles

**Integrante:** Caruk, Maria Eugenia

## 1. Objetivo y Contexto

Implementar el sistema de roles y la gestión de perfiles de usuario. Este módulo define los permisos diferenciados para cada tipo de usuario (Administrador, Organizador, Disertante, Participante) y permite a cada usuario ver y editar su información personal.

**Dependencias:** Spec 01 - Autenticación y Gestión de Usuarios (necesita la tabla `users`).
**Consumido por:** Spec 03 (Eventos), Spec 04 (Inscripciones), Spec 07 (Informes), Spec 08 (Acreditación).

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Visualización de roles disponibles
**Como** administrador de la plataforma, **quiero** ver el listado de roles del sistema, **para** conocer qué roles existen y poder asignarlos.

**Criterios de Aceptación:**
- CA-01: Solo el administrador puede acceder a la gestión de roles.
- CA-02: Se muestran los 4 roles del sistema: administrador, organizador, disertante, participante.
- CA-03: Cada rol muestra su nombre, descripción y cantidad de usuarios asignados.
- CA-04: Los roles del sistema no pueden eliminarse ni modificarse en su nombre clave (slug).

### HU-02: Asignación de roles a usuarios
**Como** administrador, **quiero** asignar o quitar roles a cualquier usuario, **para** gestionar los permisos del sistema.

**Criterios de Aceptación:**
- CA-01: El administrador puede buscar usuarios por nombre o email.
- CA-02: Desde el perfil de un usuario, el administrador puede agregar uno o más roles.
- CA-03: El administrador puede quitar roles (excepto que sea el único rol del usuario).
- CA-04: Un usuario puede tener múltiples roles simultáneamente.
- CA-05: Todo cambio de rol se registra con fecha y administrador responsable.

### HU-03: Visualización de perfil propio
**Como** usuario autenticado, **quiero** ver mi perfil con mi información personal y mis roles asignados, **para** conocer cómo me ve el sistema.

**Criterios de Aceptación:**
- CA-01: El perfil muestra: nombre, apellido, email, roles asignados, fecha de registro.
- CA-02: El perfil muestra estadísticas: cantidad de eventos inscritos, certificados obtenidos, eventos organizados (según rol).
- CA-03: Cualquier usuario puede ver su propio perfil.

### HU-04: Edición de perfil propio
**Como** usuario autenticado, **quiero** editar mi nombre, apellido y contraseña, **para** mantener mi información actualizada.

**Criterios de Aceptación:**
- CA-01: El usuario puede modificar su nombre y apellido.
- CA-02: El email NO es editable (es el identificador único de la cuenta).
- CA-03: El usuario puede cambiar su contraseña ingresando la actual y la nueva.
- CA-04: La nueva contraseña debe cumplir los mismos requisitos que el registro (mínimo 8 caracteres, 1 mayúscula, 1 minúscula, 1 número).
- CA-05: Al guardar cambios, se muestra mensaje de éxito.

### HU-05: Visualización de perfiles de otros usuarios
**Como** organizador de un evento, **quiero** ver el perfil básico de los participantes inscritos, **para** conocer quiénes asistirán.

**Criterios de Aceptación:**
- CA-01: El organizador puede ver nombre, apellido y email de los inscritos a SUS eventos.
- CA-02: NO se muestran datos sensibles (contraseña, tokens, etc.).
- CA-03: Un participante solo puede ver su propio perfil, no el de otros.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Roles del sistema
El sistema tiene 4 roles fijos que NO pueden eliminarse:
- `admin`: Administrador de plataforma. Acceso total.
- `organizer`: Organizador de eventos. Puede crear, editar y gestionar sus propios eventos.
- `speaker`: Disertante/Expositor. Puede ver eventos donde participa, evaluar encuestas.
- `participant`: Participante. Puede inscribirse a eventos, ver certificados, completar encuestas.

### RF-02: Relación N:M entre usuarios y roles
Un usuario puede tener múltiples roles. La relación se gestiona mediante la tabla intermedia `user_roles`.

### RF-03: Roles por defecto
- Al registrarse, todo usuario recibe automáticamente el rol `participant`.
- El primer usuario del sistema (seed) recibe el rol `admin`.

### RF-04: Historial de cambios de rol
Cada asignación o remoción de rol se registra en `role_change_log` con fecha, usuario que realizó el cambio y rol afectado.

### RN-01: Un usuario debe tener al menos un rol en todo momento.
### RN-02: Solo el administrador puede asignar roles `admin` y `organizer`.
### RN-03: Un organizador solo puede gestionar eventos que él creó.
### RN-04: Los roles del sistema (slugs) son inmutables: `admin`, `organizer`, `speaker`, `participant`.

## 4. Restricciones Técnicas Específicas

- **Relación N:M:** usar tabla intermedia `user_roles` con modelo de asociación de Sequelize.
- **Middleware de autorización:** crear función `requireRole(role)` que verifique si `req.user.roles` incluye el rol solicitado.
- **Permisos en controladores:** cada action debe verificar el rol antes de ejecutar.
- **No permitir auto-elevación:** un usuario no puede asignarse roles a sí mismo.
- **Audit trail:** registrar cambios de rol en tabla `role_change_log`.

## 5. Modelo de Datos

### Tabla: `roles`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| name | VARCHAR(100) | NO | Nombre legible (Administrador, Organizador, etc.) |
| slug | VARCHAR(50) | NO | UNIQUE, identificador (admin, organizer, speaker, participant) |
| description | TEXT | YES | Descripción del rol |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `user_roles`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| user_id | INT | NO | Foreign Key -> users.id |
| role_id | INT | NO | Foreign Key -> roles.id |
| assigned_by | INT | YES | Foreign Key -> users.id (admin que asignó) |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Tabla: `role_change_log`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| user_id | INT | NO | Foreign Key -> users.id (usuario afectado) |
| role_id | INT | NO | Foreign Key -> roles.id |
| action | ENUM('assigned', 'removed') | NO | Tipo de acción |
| performed_by | INT | NO | Foreign Key -> users.id (admin que realizó) |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
User.belongsToMany(Role, { through: 'user_roles', as: 'roles', foreignKey: 'user_id' });
Role.belongsToMany(User, { through: 'user_roles', as: 'users', foreignKey: 'role_id' });

User.hasMany(RoleChangeLog, { foreignKey: 'user_id', as: 'roleChanges' });
RoleChangeLog.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
RoleChangeLog.belongsTo(User, { foreignKey: 'performed_by', as: 'performedBy' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Crear modelo Role | Definir modelo Sequelize | `src/models/Role.js` |
| T02 | Crear modelo UserRole | Tabla intermedia para relación N:M | `src/models/UserRole.js` |
| T03 | Crear modelo RoleChangeLog | Auditoría de cambios de rol | `src/models/RoleChangeLog.js` |
| T04 | Crear migraciones | Migraciones para roles, user_roles, role_change_log | `migrations/` |
| T05 | Seed de roles | Insertar los 4 roles base del sistema | `seeders/` |
| T06 | Controller Roles - Listado | Listar todos los roles (admin only) | `src/controllers/roleController.js` |
| T07 | Controller Roles - Asignar | Asignar rol a usuario | `src/controllers/roleController.js` |
| T08 | Controller Roles - Quitar | Quitar rol a usuario | `src/controllers/roleController.js` |
| T09 | Rutas de roles | Definir rutas GET/POST | `src/routes/roles.js` |
| T10 | Middleware requireRole | Verificar rol en rutas protegidas | `src/middleware/auth.js` |
| T11 | Controller User - Perfil | Ver perfil propio | `src/controllers/userController.js` |
| T12 | Controller User - Editar perfil | Editar nombre, apellido | `src/controllers/userController.js` |
| T13 | Controller User - Cambiar contraseña | Flujo de cambio de contraseña | `src/controllers/userController.js` |
| T14 | Vistas de roles | Listado de roles, gestión | `src/views/admin/roles/` |
| T15 | Vistas de perfil | Ver perfil, editar perfil | `src/views/users/` |
| T16 | Vistas de usuarios (admin) | Listado y búsqueda de usuarios | `src/views/admin/users/` |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Seed de roles:** Verificar que los 4 roles existen en BD tras ejecutar seeders.
2. **Asignar rol admin:** Como admin, asignar rol organizer a un usuario participante. Verificar en BD y en perfil.
3. **Múltiples roles:** Asignar speaker + participant a un usuario. Verificar que ambos aparecen.
4. **Quitar rol:** Quitar rol a usuario (dejando al menos uno). Verificar que se elimina y se registra en log.
5. **No quitar último rol:** Intentar quitar el único rol de un usuario. Verificar que se rechaza.
6. **Editar perfil:** Cambiar nombre y apellido, verificar que se actualiza.
7. **Cambiar contraseña:** Cambiar con contraseña actual correcta, verificar login con nueva.
8. **Cambio de contraseña incorrecto:** Intentar cambiar con contraseña actual incorrecta, verificar error.
9. **Requiere rol:** Acceder a ruta de admin como participant, verificar 403.
10. **Historial de cambios:** Verificar que cada asignación/remoción queda en `role_change_log`.

### Tests de Seguridad
- Intentar asignarse rol admin a sí mismo (sin ser admin).
- Intentar acceder a gestión de roles sin ser admin.
- Verificar que solo el admin ve el listado completo de usuarios.
