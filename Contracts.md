# Contratos Técnicos del Proyecto

## 1. Stack Tecnológico

| Capa | Tecnología | Versión Mínima |
|---|---|---|
| Runtime | Node.js | 18.x |
| Framework Backend | Express.js | 4.x |
| Motor de Templates | EJS | 3.x |
| Base de Datos | MySQL | 8.x |
| ORM | Sequelize | 6.x |
| CSS Framework | Bootstrap | 5.x |
| Sesiones | express-session + connect-session-sequelize | latest |
| Hash de Contraseñas | bcrypt | 5.x |
| Generación PDF | pdfkit | latest |
| Generación QR | qrcode | latest |
| Envío de Emails | nodemailer | latest |

## 2. Estructura de Carpetas

```
proyecto/
├── src/
│   ├── config/
│   │   ├── database.js          # Configuración de Sequelize y conexión a BD
│   │   ├── session.js           # Configuración de sesiones
│   │   └── email.js             # Configuración de Nodemailer
│   ├── models/
│   │   ├── index.js             # Centraliza y asocia todos los modelos
│   │   ├── User.js
│   │   ├── Role.js
│   │   ├── EventType.js
│   │   ├── Event.js
│   │   ├── Registration.js
│   │   ├── Waitlist.js
│   │   ├── Certificate.js
│   │   ├── Survey.js
│   │   ├── Comment.js
│   │   └── Accreditation.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── userController.js
│   │   ├── roleController.js
│   │   ├── eventController.js
│   │   ├── eventTypeController.js
│   │   ├── registrationController.js
│   │   ├── certificateController.js
│   │   ├── surveyController.js
│   │   ├── commentController.js
│   │   ├── reportController.js
│   │   └── accreditationController.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── users.js
│   │   ├── roles.js
│   │   ├── events.js
│   │   ├── eventTypes.js
│   │   ├── registrations.js
│   │   ├── certificates.js
│   │   ├── surveys.js
│   │   ├── comments.js
│   │   ├── reports.js
│   │   └── accreditation.js
│   ├── middleware/
│   │   ├── auth.js              # Verificación de sesión y roles
│   │   ├── validation.js        # Validación de datos de entrada
│   │   └── errorHandler.js      # Manejo centralizado de errores
│   ├── views/                   # Templates EJS
│   │   ├── layouts/
│   │   │   └── main.ejs         # Layout principal con Bootstrap
│   │   ├── auth/
│   │   ├── users/
│   │   ├── events/
│   │   ├── registrations/
│   │   ├── certificates/
│   │   ├── surveys/
│   │   ├── reports/
│   │   └── partials/            # Header, footer, navbar, etc.
│   ├── public/
│   │   ├── css/
│   │   │   └── styles.css       # CSS custom (sobre Bootstrap)
│   │   ├── js/
│   │   │   └── main.js          # JavaScript del cliente
│   │   └── images/
│   ├── services/
│   │   ├── emailService.js      # Lógica de envío de emails
│   │   ├── pdfService.js        # Generación de certificados PDF
│   │   └── qrService.js         # Generación y validación de QR
│   └── utils/
│       └── helpers.js           # Funciones auxiliares
├── migrations/                  # Migraciones de Sequelize
├── seeders/                     # Datos iniciales (roles, admin, etc.)
├── tests/                       # Tests del proyecto
├── .env                         # Variables de entorno (NO commitear)
├── .env.example                 # Ejemplo de variables de entorno
├── app.js                       # Entry point de la aplicación
└── package.json
```

## 3. Convenciones de Base de Datos

### 3.1 Nomenclatura
- **Tablas:** `snake_case`, en plural. Ejemplo: `users`, `event_types`, `registrations`
- **Columnas:** `snake_case`, en singular. Ejemplo: `first_name`, `created_at`, `event_id`
- **Primary Keys:** `id` (INTEGER, AUTO_INCREMENT)
- **Foreign Keys:** `<tabla_singular>_id`. Ejemplo: `user_id`, `event_id`
- **Tablas intermedias (N:M):** `snake_case`, nombre compuesto. Ejemplo: `user_roles`

### 3.2 Columnas Estándar
Toda tabla DEBE incluir:
- `id` INT AUTO_INCREMENT PRIMARY KEY
- `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

### 3.3 Tipos de Datos
- **Strings cortos (nombres, emails):** VARCHAR(255)
- **Textos largos (descripciones, contenido):** TEXT
- **Fechas:** DATE (para fechas de evento) o DATETIME (para timestamps de acción)
- **Booleanos:** TINYINT(1)
- **Precios/decimales:** DECIMAL(10,2)
- **UUIDs (si se usan):** CHAR(36)

### 3.4 Migraciones
- Cada migración en archivo separado con timestamp: `YYYYMMDDHHMMSS-description.js`
- Nunca modificar migraciones ya ejecutadas
- Usar `npx sequelize-cli db:migrate` para aplicar migraciones

## 4. Estándares de Código

### 4.1 JavaScript/Node.js
- Uso de `const` y `let` (NUNCA `var`)
- Async/await para operaciones asíncronas (evitar callbacks y .then encadenados)
- ESLint configurado con `eslint:recommended`
- Nombres de variables en `camelCase`
- Nombres de clases y modelos en `PascalCase`
- Nombres de archivos en `camelCase.js`

### 4.2 Controladores
- Un método por acción (index, show, create, update, destroy)
- Separar lógica de negocio en `services/` cuando sea compleja
- Siempre retornar respuesta con status HTTP adecuado
- Manejar errores con try/catch y pasar al middleware de error

### 4.3 Rutas
- Rutas en `kebab-case` para URLs: `/event-types`, `/user-profile`
- RESTful: GET para leer, POST para crear, PUT/PATCH para actualizar, DELETE para eliminar
- Agrupar rutas por recurso con `express.Router()`

### 4.4 Vistas (EJS)
- Layout principal en `views/layouts/main.ejs`
- Partials reutilizables en `views/partials/`
- Usar Bootstrap 5 para toda la UI
- Formularios con validación tanto del lado del servidor como del cliente

### 4.5 Sesiones y Autenticación
- Usar `express-session` con store en base de datos (`connect-session-sequelize`)
- Contraseñas hasheadas con `bcrypt` (salt rounds: 10)
- Middleware de autenticación en `middleware/auth.js`
- Middleware de autorización por rol: `requireRole('admin')`, `requireRole('organizer')`

## 5. Formato de Respuestas

### 5.1 Respuestas Exitosas (JSON para API)
```json
{
  "success": true,
  "data": {},
  "message": "Operación realizada con éxito"
}
```

### 5.2 Respuestas de Error
```json
{
  "success": false,
  "message": "Descripción del error",
  "errors": [
    { "field": "email", "message": "El email ya está registrado" }
  ]
}
```

### 5.3 Status HTTP
- `200` - OK (lectura exitosa)
- `201` - Created (recurso creado)
- `302` - Redirect (redirecciones web)
- `400` - Bad Request (validación fallida)
- `401` - Unauthorized (no autenticado)
- `403` - Forbidden (sin permisos)
- `404` - Not Found (recurso no existe)
- `500` - Internal Server Error (error del servidor)

## 6. Variables de Entorno (.env)

```env
# Servidor
PORT=3000
NODE_ENV=development

# Base de Datos
DB_HOST=localhost
DB_PORT=3306
DB_NAME=eventos_academicos
DB_USER=root
DB_PASSWORD=

# Sesión
SESSION_SECRET=cambiar_esto_en_produccion

# Email (Nodemailer)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=tu_email@gmail.com
EMAIL_PASS=tu_app_password

# URL base
BASE_URL=http://localhost:3000
```

## 7. Git Workflow

- **Rama principal:** `main` (protegida, no hacer push directo)
- **Ramas de feature:** `feature/nombre-del-modulo`
- **Commits:** convencionales - `feat: agregar modelo de eventos`, `fix: corregir validación de email`
- **Antes de commitear:** verificar que `npm run lint` no tenga errores
- **NO commitear:** `.env`, `node_modules/`, archivos generados

## 8. Scripts del package.json

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "lint": "eslint src/",
    "db:migrate": "npx sequelize-cli db:migrate",
    "db:seed": "npx sequelize-cli db:seed:all",
    "db:reset": "npx sequelize-cli db:drop && npx sequelize-cli db:create && npx sequelize-cli db:migrate && npx sequelize-cli db:seed:all"
  }
}
```
