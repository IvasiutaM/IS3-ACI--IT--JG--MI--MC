# Spec 01 - Autenticación y Gestión de Usuarios

**Integrante:** Caruk, Maria Eugenia

## 1. Objetivo y Contexto

Implementar el sistema de autenticación y gestión de cuentas de usuario. Este módulo es la puerta de entrada al sistema y es prerrequisito para todos los demás módulos. Permite a cualquier persona crear una cuenta, verificar su email, iniciar sesión, recuperar su contraseña y gestionar su perfil básico.

**Dependencias:** Ninguna (módulo base).
**Consumido por:** Todos los módulos del sistema.

## 2. Historias de Usuario y Criterios de Aceptación

### HU-01: Registro de usuario
**Como** visitante del sitio, **quiero** crear una cuenta con mi email y contraseña, **para** poder inscribirme a eventos y acceder a mis certificados.

**Criterios de Aceptación:**
- CA-01: El formulario de registro solicita: nombre, apellido, email, contraseña, confirmar contraseña.
- CA-02: El email debe tener formato válido y no estar registrado previamente.
- CA-03: La contraseña debe tener mínimo 8 caracteres, al menos 1 mayúscula, 1 minúscula, 1 número.
- CA-04: Al registrarse exitosamente, se envía un email con enlace de verificación.
- CA-05: El usuario NO puede iniciar sesión hasta verificar su email.
- CA-06: Se muestra mensaje de éxito con indicación de revisar el email.

### HU-02: Verificación de email
**Como** usuario recién registrado, **quiero** verificar mi email haciendo clic en un enlace, **para** activar mi cuenta.

**Criterios de Aceptación:**
- CA-01: El enlace de verificación contiene un token único con expiración de 24 horas.
- CA-02: Al hacer clic en el enlace válido, la cuenta se activa y se redirige al login con mensaje de éxito.
- CA-03: Si el token expiró, se muestra mensaje de error con opción de reenviar verificación.
- CA-04: Si el token es inválido, se muestra error genérico.

### HU-03: Inicio de sesión
**Como** usuario registrado, **quiero** iniciar sesión con mi email y contraseña, **para** acceder a mis funcionalidades.

**Criterios de Aceptación:**
- CA-01: El formulario de login solicita email y contraseña.
- CA-02: Si las credenciales son correctas y el email está verificado, se crea la sesión y se redirige al dashboard.
- CA-03: Si el email no está verificado, se muestra mensaje indicando que debe verificar su email, con opción de reenviar enlace.
- CA-04: Si las credenciales son incorrectas, se muestra mensaje genérico "Email o contraseña incorrectos".
- CA-05: Tras 5 intentos fallidos consecutivos, se bloquea la cuenta por 15 minutos.
- CA-06 (OWASP Rate Limiting): El bloqueo tras 5 intentos fallidos debe implementarse a nivel de IP y a nivel de usuario (email) de forma independiente para mitigar ataques de fuerza bruta distribuida y Credential Stuffing.
- CA-07 (OWASP Session Management): Al autenticarse exitosamente, el sistema debe regenerar el ID de sesión (Session Fixation protection) antes de redirigir al dashboard.

### HU-04: Cierre de sesión
**Como** usuario autenticado, **quiero** cerrar mi sesión, **para** proteger mi cuenta.

**Criterios de Aceptación:**
- CA-01: Al cerrar sesión se destruye la sesión del servidor y las cookies.
- CA-02: Se redirige al home con mensaje de despedida.
- CA-03: No se puede volver atrás con el navegador del browser y seguir autenticado.

### HU-05: Recuperación de contraseña
**Como** usuario que olvidó su contraseña, **quiero** restablecerla mediante un enlace enviado a mi email, **para** recuperar el acceso a mi cuenta.

**Criterios de Aceptación:**
- CA-01: El formulario de recuperación solicita el email registrado.
- CA-02: Se envía un email con un enlace de restablecimiento con token válido por 1 hora.
- CA-03: El formulario de nueva contraseña solicita: nueva contraseña y confirmación (mismas reglas que registro).
- CA-04: Si el token es válido, se actualiza la contraseña y se redirige al login.
- CA-05: Si el token expiró o es inválido, se muestra error.

### HU-06: Reenvío de verificación
**Como** usuario con email no verificado, **quiero** solicitar un nuevo enlace de verificación, **para** activar mi cuenta si el enlace original expiró.

**Criterios de Aceptación:**
- CA-01: El usuario puede solicitar un nuevo enlace desde la pantalla de login.
- CA-02: Se envía un nuevo email con token válido por 24 horas.
- CA-03: Se limita a 3 solicitudes por hora para evitar abuso.

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-01: Registro
- El sistema debe crear un nuevo registro en la tabla `users` con `is_verified = false`.
- Se asigna automáticamente el rol de `participante` al registrarse (vía tabla intermedia `user_roles`).
- Las contraseñas NUNCA se almacenan en texto plano; se usa bcrypt con salt rounds de 10.

### RF-02: Verificación
- El token de verificación se almacena en la tabla `email_tokens` con `type = 'verification'`.
- Un token solo es válido una vez; después de usado, se marca como `used = true`.

### RF-03: Sesiones
- La duración de la sesión es de 24 horas de inactividad.
- Se usa `express-session` con store en base de datos.

### RF-04: Recuperación de contraseña
- El token de recuperación se almacena en `email_tokens` con `type = 'password_reset'`.
- Al cambiar la contraseña exitosamente, se invalidan TODOS los tokens activos del usuario.

### RN-01: Un email solo puede registrarse una vez.
### RN-02: Un usuario no verificado no puede acceder a ninguna funcionalidad protegida.
### RN-03: Tras 5 intentos fallidos de login, la cuenta se bloquea por 15 minutos.
### RN-04: Máximo 3 reenvíos de verificación por hora por email.

## 4. Restricciones Técnicas Específicas

- **Hash:** bcrypt con `saltRounds = 10`
- **Tokens:** usar `crypto.randomBytes(32).toString('hex')` para generar tokens seguros
- **Sesiones:** store en base de datos mediante `connect-session-sequelize`
- **Email:** usar Nodemailer con transporter configurado desde `.env`
- **Middleware:** crear middleware `isAuthenticated` que verifique `req.session.userId`
- **CSRF:** proteger formularios con token CSRF
- **Rate limiting:** para login (5 intentos/hora por IP) y reenvío de email (3/hora por email)

## 5. Modelo de Datos

### Tabla: `users`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| first_name | VARCHAR(100) | NO | Nombre |
| last_name | VARCHAR(100) | NO | Apellido |
| email | VARCHAR(255) | NO | UNIQUE, índice |
| password_hash | VARCHAR(255) | NO | Hash bcrypt |
| is_verified | TINYINT(1) | NO | DEFAULT 0 |
| failed_login_attempts | INT | NO | DEFAULT 0 |
| locked_until | DATETIME | YES | Fecha de desbloqueo |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

### Tabla: `email_tokens`
| Columna | Tipo | Nullable | Descripción |
|---|---|---|---|
| id | INT AUTO_INCREMENT | NO | Primary Key |
| user_id | INT | NO | Foreign Key -> users.id |
| token | VARCHAR(64) | NO | UNIQUE, índice |
| type | ENUM('verification', 'password_reset') | NO | Tipo de token |
| expires_at | DATETIME | NO | Fecha de expiración |
| used | TINYINT(1) | NO | DEFAULT 0 |
| created_at | TIMESTAMP | NO | DEFAULT CURRENT_TIMESTAMP |

### Relaciones Sequelize
```javascript
User.hasMany(EmailToken, { foreignKey: 'user_id', as: 'tokens' });
EmailToken.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
```

## 6. Plan de Tareas

| # | Tarea | Descripción | Archivos |
|---|---|---|---|
| T01 | Configurar sesión | Configurar express-session con store en BD | `src/config/session.js`, `app.js` |
| T02 | Crear modelo User | Definir modelo Sequelize con validaciones | `src/models/User.js` |
| T03 | Crear modelo EmailToken | Definir modelo para tokens de verificación y reset | `src/models/EmailToken.js` |
| T04 | Crear migraciones | Migraciones para users y email_tokens | `migrations/` |
| T05 | Configurar email | Configurar Nodemailer transporter | `src/config/email.js`, `src/services/emailService.js` |
| T06 | Controller Auth - Registro | Lógica de registro, hash de password, envío de email | `src/controllers/authController.js` |
| T07 | Controller Auth - Verificación | Lógica de verificación de token | `src/controllers/authController.js` |
| T08 | Controller Auth - Login | Lógica de login, validación, bloqueo por intentos | `src/controllers/authController.js` |
| T09 | Controller Auth - Logout | Destrucción de sesión | `src/controllers/authController.js` |
| T10 | Controller Auth - Recuperación | Flujo completo de forgot/reset password | `src/controllers/authController.js` |
| T11 | Middleware isAuthenticated | Verificar sesión activa en rutas protegidas | `src/middleware/auth.js` |
| T12 | Rutas de auth | Definir todas las rutas GET/POST | `src/routes/auth.js` |
| T13 | Vistas de auth | Login, registro, forgot password, reset password, verificación | `src/views/auth/*.ejs` |
| T14 | Seeders | Seed de roles iniciales y usuario admin | `seeders/` |
| T15 | Validaciones de formulario | Validación cliente y servidor | `src/middleware/validation.js`, vistas |

## 7. Estrategia de Verificación

### Tests Manuales
1. **Registro exitoso:** Registrar un usuario nuevo, verificar que se crea en BD con `is_verified=0`, verificar que llega el email.
2. **Verificación de email:** Clic en enlace, verificar que `is_verified=1`, poder hacer login.
3. **Login con credenciales correctas:** Verificar redirección al dashboard.
4. **Login con credenciales incorrectas:** Verificar mensaje de error genérico.
5. **Bloqueo por intentos:** Intentar login 5 veces con contraseña incorrecta, verificar bloqueo por 15 minutos.
6. **Login sin verificar email:** Verificar que se muestra mensaje de verificación pendiente.
7. **Recuperación de contraseña:** Solicitar reset, clic en enlace, cambiar contraseña, verificar login con nueva contraseña.
8. **Token expirado:** Esperar a que expire token de verificación (o modificar `expires_at` en BD), verificar mensaje de error.
9. **Reenvío de verificación:** Solicitar reenvío, verificar que llega nuevo email.
10. **Logout:** Cerrar sesión, verificar que no se puede acceder a rutas protegidas.

### Tests de Seguridad
- Intentar acceder a rutas protegidas sin sesión.
- Intentar inyección SQL en campos de formulario.
- Verificar que las contraseñas están hasheadas en BD.
- Verificar que los tokens no son predecibles.
- Verificar CSRF en formularios.
