# AGENTS.md - Guía para Agentes de IA

## Descripción General del Proyecto

**IDURAR** es un sistema ERP/CRM de código abierto construido sobre el stack MERN (MongoDB, Express.js, React.js, Node.js) con Ant Design (AntD) y Redux. El proyecto está licenciado bajo GNU AGPL v3.0 (Fair-Code).

### Información del Proyecto
- **Versión**: 4.1.0
- **Node.js requerido**: 20.9.0
- **npm requerido**: 10.2.4
- **Puerto Backend**: 8888 (por defecto)
- **Puerto Frontend**: 3000 (por defecto)

---

## Arquitectura del Sistema

```
idurar-erp/
├── backend/                    # API REST con Express.js
│   └── src/
│       ├── app.js             # Configuración principal de Express
│       ├── server.js          # Punto de entrada del servidor
│       ├── controllers/       # Lógica de negocio
│       ├── models/            # Esquemas de Mongoose
│       ├── routes/            # Definición de rutas API
│       ├── middlewares/       # Middlewares personalizados
│       ├── handlers/          # Manejadores de errores
│       ├── utils/             # Utilidades generales
│       ├── setup/             # Scripts de configuración inicial
│       ├── locale/            # Internacionalización
│       ├── pdf/               # Plantillas Pug para PDF
│       └── emailTemplate/     # Plantillas de email
│
├── frontend/                   # SPA con React.js
│   └── src/
│       ├── main.jsx           # Punto de entrada
│       ├── RootApp.jsx        # Componente raíz
│       ├── apps/              # Aplicaciones principales
│       ├── components/        # Componentes reutilizables
│       ├── modules/           # Módulos funcionales
│       ├── pages/             # Páginas de la aplicación
│       ├── redux/             # Estado global (Redux Toolkit)
│       ├── context/           # Contextos de React
│       ├── hooks/             # Hooks personalizados
│       ├── forms/             # Formularios
│       ├── layout/            # Layouts de la aplicación
│       ├── router/            # Configuración de rutas
│       ├── auth/              # Servicios de autenticación
│       ├── request/           # Cliente HTTP (Axios)
│       ├── locale/            # Internacionalización (44 idiomas)
│       ├── settings/          # Configuraciones
│       └── style/             # Estilos CSS
│
├── doc/                        # Documentación traducida
└── features/                   # Descripciones de características (40+ idiomas)
```

---

## Stack Tecnológico

### Backend
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| Node.js | 20.9.0 | Runtime |
| Express.js | 4.18.2 | Framework web |
| MongoDB/Mongoose | 8.1.1 | Base de datos |
| JWT | 9.0.2 | Autenticación |
| Joi | 17.11.0 | Validación |
| bcryptjs | 2.4.3 | Hash de contraseñas |
| Pug | 3.0.2 | Plantillas PDF |
| html-pdf | 3.0.1 | Generación de PDF |
| Resend | 2.0.0 | Envío de emails |
| OpenAI | 4.27.0 | Integración IA |
| AWS S3 | 3.509.0 | Almacenamiento en la nube |

### Frontend
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| React.js | 18.3.1 | Framework UI |
| Redux Toolkit | 2.2.1 | Estado global |
| React Router | 6.22.0 | Enrutamiento |
| Ant Design | 5.14.1 | Componentes UI |
| Axios | 1.6.2 | Cliente HTTP |
| Vite | 5.4.8 | Bundler/Dev Server |
| Day.js | 1.11.10 | Manejo de fechas |

---

## Modelos de Datos (MongoDB)

### Modelos de Aplicación (`backend/src/models/appModels/`)

#### Client (Cliente)
```javascript
{
  removed: Boolean,        // Soft delete
  enabled: Boolean,        // Estado activo
  name: String,           // Nombre (requerido)
  phone: String,
  country: String,
  address: String,
  email: String,
  createdBy: ObjectId,    // Referencia a Admin
  assigned: ObjectId,     // Admin asignado
  created: Date,
  updated: Date
}
```

#### Invoice (Factura)
```javascript
{
  removed: Boolean,
  createdBy: ObjectId,     // Admin que creó
  number: Number,          // Número de factura
  year: Number,
  date: Date,
  expiredDate: Date,
  client: ObjectId,        // Referencia a Client (autopopulate)
  items: [{
    itemName: String,
    description: String,
    quantity: Number,
    price: Number,
    total: Number
  }],
  taxRate: Number,
  subTotal: Number,
  taxTotal: Number,
  total: Number,
  currency: String,
  credit: Number,
  discount: Number,
  payment: [ObjectId],     // Referencias a Payment
  paymentStatus: String,   // 'unpaid' | 'paid' | 'partially'
  status: String,          // 'draft' | 'pending' | 'sent' | 'refunded' | 'cancelled' | 'on hold'
  recurring: String,       // 'daily' | 'weekly' | 'monthly' | 'annually' | 'quarter'
  notes: String,
  pdf: String,
  files: [{ id, name, path, description, isPublic }]
}
```

#### Quote (Cotización)
- Estructura similar a Invoice
- Puede convertirse a Invoice

#### Payment (Pago)
- Referencia a Invoice
- Información de pago

#### PaymentMode (Modo de Pago)
- Métodos de pago disponibles

#### Taxes (Impuestos)
- Configuración de tasas impositivas

### Modelos Core (`backend/src/models/coreModels/`)

#### Admin
```javascript
{
  removed: Boolean,
  enabled: Boolean,
  email: String,          // Único, lowercase
  name: String,
  surname: String,
  photo: String,
  role: String,           // 'owner'
  created: Date
}
```

#### AdminPassword
- Almacena contraseñas hasheadas
- Salt único por usuario

#### Setting
- Configuraciones del sistema
- Key-value pairs

---

## Arquitectura del Backend

### Sistema de Rutas

#### Rutas de Autenticación (`/api`)
```
POST /api/login           - Iniciar sesión
POST /api/logout          - Cerrar sesión
POST /api/forgetpassword  - Recuperar contraseña
POST /api/resetpassword   - Restablecer contraseña
```

#### Rutas Core (`/api`) - Requieren autenticación
```
GET    /api/admin/read/:id
PATCH  /api/admin/password-update/:id
PATCH  /api/admin/profile/password
PATCH  /api/admin/profile/update

POST   /api/setting/create
GET    /api/setting/read/:id
PATCH  /api/setting/update/:id
GET    /api/setting/search
GET    /api/setting/list
GET    /api/setting/listAll
GET    /api/setting/filter
GET    /api/setting/readBySettingKey/:settingKey
GET    /api/setting/listBySettingKey
PATCH  /api/setting/updateBySettingKey/:settingKey
PATCH  /api/setting/updateManySetting
```

#### Rutas de Aplicación (`/api`) - CRUD Genérico
Para cada entidad (invoice, quote, payment, client, taxes, paymentmode):
```
POST   /api/{entity}/create
GET    /api/{entity}/read/:id
PATCH  /api/{entity}/update/:id
DELETE /api/{entity}/delete/:id
GET    /api/{entity}/search
GET    /api/{entity}/list
GET    /api/{entity}/listAll
GET    /api/{entity}/filter
GET    /api/{entity}/summary

# Solo para invoice, quote, payment:
POST   /api/{entity}/mail

# Solo para quote:
GET    /api/quote/convert/:id
```

### Controlador CRUD Genérico

El sistema usa un patrón factory para crear controladores CRUD:

```javascript
// backend/src/controllers/middlewaresControllers/createCRUDController/index.js
const createCRUDController = (modelName) => {
  const Model = mongoose.model(modelName);
  return {
    create: (req, res) => create(Model, req, res),
    read: (req, res) => read(Model, req, res),
    update: (req, res) => update(Model, req, res),
    delete: (req, res) => remove(Model, req, res),
    list: (req, res) => paginatedList(Model, req, res),
    listAll: (req, res) => listAll(Model, req, res),
    search: (req, res) => search(Model, req, res),
    filter: (req, res) => filter(Model, req, res),
    summary: (req, res) => summary(Model, req, res),
  };
};
```

### Auto-descubrimiento de Modelos

Los modelos se registran automáticamente usando glob:

```javascript
// backend/src/models/utils/index.js
const appModelsFiles = globSync('./src/models/appModels/**/*.js');
// Genera automáticamente: routesList, controllersList, modelsList
```

---

## Arquitectura del Frontend

### Estado Global (Redux)

```javascript
// frontend/src/redux/rootReducer.js
const rootReducer = combineReducers({
  auth: authReducer,           // Autenticación
  crud: crudReducer,           // Operaciones CRUD genéricas
  erp: erpReducer,             // Estado específico ERP
  adavancedCrud: adavancedCrudReducer,
  settings: settingsReducer,   // Configuraciones
});
```

### Estructura de Módulos

```
modules/
├── AuthModule/          # Login, registro, recuperación
├── CrudModule/          # CRUD genérico
├── DashboardModule/     # Panel principal
├── ErpPanelModule/      # Panel ERP (invoices, quotes, etc.)
├── InvoiceModule/       # Gestión de facturas
├── QuoteModule/         # Gestión de cotizaciones
├── PaymentModule/       # Gestión de pagos
├── ProfileModule/       # Perfil de usuario
└── SettingModule/       # Configuraciones
```

### Contextos de React

```
context/
├── adavancedCrud/       # CRUD avanzado
├── appContext/          # Contexto general de app
├── crud/                # Operaciones CRUD
├── erp/                 # Contexto ERP
└── profileContext/      # Perfil de usuario
```

### Rutas del Frontend

```javascript
// frontend/src/router/routes.jsx
{
  '/': Dashboard,
  '/customer': Customer,
  '/invoice': Invoice,
  '/invoice/create': InvoiceCreate,
  '/invoice/read/:id': InvoiceRead,
  '/invoice/update/:id': InvoiceUpdate,
  '/invoice/pay/:id': InvoiceRecordPayment,
  '/quote': Quote,
  '/quote/create': QuoteCreate,
  '/quote/read/:id': QuoteRead,
  '/quote/update/:id': QuoteUpdate,
  '/payment': Payment,
  '/payment/read/:id': PaymentRead,
  '/payment/update/:id': PaymentUpdate,
  '/settings': Settings,
  '/payment/mode': PaymentMode,
  '/taxes': Taxes,
  '/profile': Profile,
  '/about': About,
  '/logout': Logout
}
```

---

## Patrones de Desarrollo

### Convenciones de Código

1. **Nombres de archivos**:
   - Modelos: PascalCase (`Invoice.js`, `Client.js`)
   - Controladores: camelCase con sufijo (`invoiceController/`)
   - Componentes React: PascalCase (`DataTable.jsx`)

2. **Estructura de controladores de entidad**:
   ```
   entityController/
   ├── index.js          # Exportación principal
   ├── create.js         # Crear
   ├── read.js           # Leer
   ├── update.js         # Actualizar
   ├── remove.js         # Eliminar (soft delete)
   ├── summary.js        # Resumen/estadísticas
   ├── paginatedList.js  # Lista paginada
   └── sendMail.js       # Envío de email
   ```

3. **Soft Delete**: Todos los modelos usan `removed: Boolean` en lugar de eliminar registros.

4. **Autopopulate**: Los modelos relacionados usan `mongoose-autopopulate` para poblado automático.

### Manejo de Errores

```javascript
// backend/src/handlers/errorHandlers.js
exports.catchErrors = (fn) => {
  return function (req, res, next) {
    return fn(req, res, next).catch(next);
  };
};

exports.notFound = (req, res, next) => { /* 404 handler */ };
exports.productionErrors = (err, req, res, next) => { /* Error handler */ };
```

### Autenticación JWT

- Token almacenado en cookies HttpOnly
- Middleware `isValidAuthToken` para rutas protegidas
- Renovación automática de tokens

---

## Configuración Inicial

### Variables de Entorno (`.env`)

```env
DATABASE=mongodb://...       # URI de MongoDB
PORT=8888                    # Puerto del servidor
JWT_SECRET=...               # Secreto JWT
OPENAI_API_KEY=...          # API Key de OpenAI (opcional)
```

### Script de Setup

```bash
cd backend
npm run setup
```

Este script crea:
1. Usuario admin (`admin@admin.com` / `admin123`)
2. Configuraciones por defecto desde `setup/defaultSettings/`
3. Impuesto por defecto (0%)
4. Modo de pago por defecto

### Configuraciones por Defecto

```
setup/defaultSettings/
├── appSettings.json
├── clientSettings.json
├── companySettings.json
├── financeSettings.json
├── invoiceSettings.json
├── moneyFormatSettings.json
└── quoteSettings.json
```

---

## Funcionalidades Principales

### 1. Gestión de Facturas
- Creación, edición, eliminación
- Múltiples estados (draft, pending, sent, paid)
- Generación de PDF
- Envío por email
- Tracking de pagos parciales
- Conversión desde cotizaciones

### 2. Gestión de Cotizaciones
- CRUD completo
- Conversión a factura
- Estados personalizables
- Generación de PDF

### 3. Gestión de Clientes
- Información de contacto
- Historial de transacciones
- Asignación a administradores

### 4. Gestión de Pagos
- Registro de pagos
- Múltiples modos de pago
- Tracking de estado

### 5. Configuraciones
- Información de empresa
- Formatos de moneda
- Tasas de impuestos
- Modos de pago

---

## Internacionalización

El sistema soporta 44+ idiomas:
- Traducciones en `frontend/src/locale/`
- Archivos de features en 40+ idiomas en `features/`

---

## Consideraciones para Agentes

### Al modificar Backend:
1. Usar el patrón `createCRUDController` para nuevas entidades
2. Registrar nuevos modelos en `appModels/`
3. Las rutas se generan automáticamente desde los modelos
4. Mantener el patrón de soft delete
5. Usar `catchErrors` wrapper en controladores async

### Al modificar Frontend:
1. Seguir patrón de Redux existente (actions, types, reducer, selectors)
2. Usar componentes de Ant Design
3. Lazy loading para páginas
4. Usar hooks personalizados existentes
5. Mantener consistencia con el patrón de módulos

### Mejores Prácticas:
1. **No actualizar dependencias** sin revisión completa
2. Seguir el formato de commits: `feat:`, `fix:`, etc.
3. Crear branch desde `dev`, no desde `main`
4. Nombrar branches: `features/nombre` o `issues/descripcion`
5. Mantener compatibilidad con Node.js 20.9.0

---

## Comandos Útiles

### Backend
```bash
npm run dev      # Desarrollo con nodemon
npm run start    # Producción
npm run setup    # Configuración inicial
npm run reset    # Resetear base de datos
npm run upgrade  # Actualizar configuraciones
```

### Frontend
```bash
npm run dev         # Desarrollo con Vite
npm run build       # Build de producción
npm run preview     # Vista previa del build
npm run lint        # ESLint
```

---

## Seguridad

- Reporte de vulnerabilidades: GitHub Security Advisors
- No crear issues públicos para problemas de seguridad
- Contacto: hello@idurarapp.com

---

## Licencia

GNU Affero General Public License v3.0 (AGPL-3.0)

**Importante**: No se permite ofrecer IDURAR como SaaS a terceros sin liberar el código fuente modificado.

---

## Recursos

- **Demo**: https://www.idurarapp.com/demo-erp-crm/
- **Credenciales demo**: `admin@admin.com` / `admin123`
- **GitHub**: https://github.com/idurar/idurar-erp-crm
- **Documentación**: https://www.idurarapp.com

