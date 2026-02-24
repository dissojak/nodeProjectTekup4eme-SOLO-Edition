# Step 12 — Adem: Add Swagger Documentation

**Commit message:** `Adem: add swagger documentation`

---

## What to do

Add Swagger/OpenAPI documentation to all API endpoints. Uses `swagger-jsdoc` for auto-generating docs from JSDoc comments and `swagger-ui-express` to serve the UI at `/api-docs`.

## Install dependencies

```bash
npm install swagger-jsdoc swagger-ui-express
```

## Files to create/update

### 1. `config/swagger.js`

```js
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Recouvra+ API',
      version: '1.0.0',
      description: 'API de gestion du recouvrement - Debt Collection Management API',
      contact: {
        name: 'Adem & Baha',
      },
    },
    servers: [
      {
        url: 'http://localhost:5000',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        cookieAuth: {
          type: 'apiKey',
          in: 'cookie',
          name: 'jwt',
        },
      },
      schemas: {
        User: {
          type: 'object',
          properties: {
            _id: { type: 'string', example: '507f1f77bcf86cd799439011' },
            name: { type: 'string', example: 'Adem' },
            email: { type: 'string', example: 'adem@test.com' },
            role: { type: 'string', enum: ['agent', 'manager', 'admin'], example: 'agent' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Client: {
          type: 'object',
          properties: {
            _id: { type: 'string' },
            name: { type: 'string', example: 'Company ABC' },
            email: { type: 'string', example: 'contact@abc.com' },
            phone: { type: 'string', example: '12345678' },
            address: { type: 'string', example: '123 Main St' },
            createdBy: { type: 'string' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Invoice: {
          type: 'object',
          properties: {
            _id: { type: 'string' },
            invoiceNumber: { type: 'string', example: 'INV-001' },
            client: { type: 'string' },
            amount: { type: 'number', example: 1500 },
            amountPaid: { type: 'number', example: 0 },
            dueDate: { type: 'string', format: 'date' },
            status: { type: 'string', enum: ['unpaid', 'partially_paid', 'paid', 'overdue'] },
            createdBy: { type: 'string' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Payment: {
          type: 'object',
          properties: {
            _id: { type: 'string' },
            invoice: { type: 'string' },
            amount: { type: 'number', example: 500 },
            paymentDate: { type: 'string', format: 'date-time' },
            paymentMethod: { type: 'string', enum: ['cash', 'check', 'transfer'] },
            note: { type: 'string' },
            recordedBy: { type: 'string' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        RecoveryAction: {
          type: 'object',
          properties: {
            _id: { type: 'string' },
            invoice: { type: 'string' },
            client: { type: 'string' },
            actionType: { type: 'string', enum: ['phone_call', 'email', 'letter', 'visit', 'legal'] },
            note: { type: 'string' },
            result: { type: 'string' },
            actionDate: { type: 'string', format: 'date-time' },
            performedBy: { type: 'string' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Error: {
          type: 'object',
          properties: {
            message: { type: 'string' },
            stack: { type: 'string' },
          },
        },
      },
    },
    security: [{ cookieAuth: [] }],
  },
  apis: ['./routes/*.js'],
};

const swaggerSpec = swaggerJsdoc(options);

module.exports = swaggerSpec;
```

### 2. Update `app.js` — add Swagger UI

Add these lines near the top (after other requires):

```js
const swaggerUi = require('swagger-ui-express');
const swaggerSpec = require('./config/swagger');
```

Add this BEFORE route registrations:

```js
// Swagger documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

### 3. Add JSDoc comments to ALL route files

You need to add Swagger JSDoc comments above each route. Here are the annotations for each route file:

#### `routes/auth.js` — add these comments above each route:

```js
/**
 * @swagger
 * tags:
 *   name: Auth
 *   description: Authentication and authorization
 */

/**
 * @swagger
 * /api/auth/register:
 *   post:
 *     summary: Register a new user
 *     tags: [Auth]
 *     security: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *               - password
 *             properties:
 *               name:
 *                 type: string
 *                 example: Adem
 *               email:
 *                 type: string
 *                 example: adem@test.com
 *               password:
 *                 type: string
 *                 example: "123456"
 *               role:
 *                 type: string
 *                 enum: [agent, manager, admin]
 *     responses:
 *       201:
 *         description: User registered successfully
 *       400:
 *         description: Validation error or user already exists
 */

/**
 * @swagger
 * /api/auth/login:
 *   post:
 *     summary: Login user
 *     tags: [Auth]
 *     security: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - email
 *               - password
 *             properties:
 *               email:
 *                 type: string
 *               password:
 *                 type: string
 *     responses:
 *       200:
 *         description: Login successful (JWT cookie set)
 *       401:
 *         description: Invalid credentials
 */

/**
 * @swagger
 * /api/auth/logout:
 *   post:
 *     summary: Logout user (clear cookie)
 *     tags: [Auth]
 *     responses:
 *       200:
 *         description: Logged out successfully
 */

/**
 * @swagger
 * /api/auth/me:
 *   get:
 *     summary: Get current user profile
 *     tags: [Auth]
 *     responses:
 *       200:
 *         description: Current user data
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 */
```

#### `routes/clients.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Clients
 *   description: Client management
 */

/**
 * @swagger
 * /api/clients:
 *   get:
 *     summary: Get all clients
 *     tags: [Clients]
 *     responses:
 *       200:
 *         description: List of clients
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Client'
 *   post:
 *     summary: Create a new client
 *     tags: [Clients]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               phone:
 *                 type: string
 *               address:
 *                 type: string
 *     responses:
 *       201:
 *         description: Client created
 *       400:
 *         description: Validation error
 */

/**
 * @swagger
 * /api/clients/{id}:
 *   get:
 *     summary: Get client by ID
 *     tags: [Clients]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Client data
 *       404:
 *         description: Client not found
 *   put:
 *     summary: Update a client
 *     tags: [Clients]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               phone:
 *                 type: string
 *               address:
 *                 type: string
 *     responses:
 *       200:
 *         description: Client updated
 *       404:
 *         description: Client not found
 *   delete:
 *     summary: Delete a client
 *     tags: [Clients]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Client deleted
 *       404:
 *         description: Client not found
 */
```

#### `routes/invoices.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Invoices
 *   description: Invoice management
 */

/**
 * @swagger
 * /api/invoices:
 *   get:
 *     summary: Get all invoices
 *     tags: [Invoices]
 *     responses:
 *       200:
 *         description: List of invoices
 *   post:
 *     summary: Create a new invoice
 *     tags: [Invoices]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - invoiceNumber
 *               - client
 *               - amount
 *               - dueDate
 *             properties:
 *               invoiceNumber:
 *                 type: string
 *               client:
 *                 type: string
 *                 description: Client ID
 *               amount:
 *                 type: number
 *               dueDate:
 *                 type: string
 *                 format: date
 *     responses:
 *       201:
 *         description: Invoice created
 */

/**
 * @swagger
 * /api/invoices/client/{clientId}:
 *   get:
 *     summary: Get invoices by client
 *     tags: [Invoices]
 *     parameters:
 *       - in: path
 *         name: clientId
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: List of invoices for client
 */

/**
 * @swagger
 * /api/invoices/{id}:
 *   get:
 *     summary: Get invoice by ID
 *     tags: [Invoices]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Invoice data
 *       404:
 *         description: Invoice not found
 *   put:
 *     summary: Update an invoice
 *     tags: [Invoices]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               invoiceNumber:
 *                 type: string
 *               amount:
 *                 type: number
 *               dueDate:
 *                 type: string
 *                 format: date
 *               status:
 *                 type: string
 *                 enum: [unpaid, partially_paid, paid, overdue]
 *     responses:
 *       200:
 *         description: Invoice updated
 *   delete:
 *     summary: Delete an invoice
 *     tags: [Invoices]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Invoice deleted
 */
```

#### `routes/payments.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Payments
 *   description: Payment recording and tracking
 */

/**
 * @swagger
 * /api/payments:
 *   get:
 *     summary: Get all payments
 *     tags: [Payments]
 *     responses:
 *       200:
 *         description: List of payments
 *   post:
 *     summary: Record a new payment
 *     tags: [Payments]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - invoice
 *               - amount
 *               - paymentMethod
 *             properties:
 *               invoice:
 *                 type: string
 *                 description: Invoice ID
 *               amount:
 *                 type: number
 *               paymentMethod:
 *                 type: string
 *                 enum: [cash, check, transfer]
 *               paymentDate:
 *                 type: string
 *                 format: date-time
 *               note:
 *                 type: string
 *     responses:
 *       201:
 *         description: Payment recorded, invoice status updated
 */

/**
 * @swagger
 * /api/payments/invoice/{invoiceId}:
 *   get:
 *     summary: Get payments by invoice
 *     tags: [Payments]
 *     parameters:
 *       - in: path
 *         name: invoiceId
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: List of payments for invoice
 */
```

#### `routes/recoveryActions.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Recovery Actions
 *   description: Debt recovery action tracking
 */

/**
 * @swagger
 * /api/recovery-actions:
 *   get:
 *     summary: Get all recovery actions
 *     tags: [Recovery Actions]
 *     responses:
 *       200:
 *         description: List of recovery actions
 *   post:
 *     summary: Create a new recovery action
 *     tags: [Recovery Actions]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - invoice
 *               - client
 *               - actionType
 *             properties:
 *               invoice:
 *                 type: string
 *               client:
 *                 type: string
 *               actionType:
 *                 type: string
 *                 enum: [phone_call, email, letter, visit, legal]
 *               note:
 *                 type: string
 *               result:
 *                 type: string
 *               actionDate:
 *                 type: string
 *                 format: date-time
 *     responses:
 *       201:
 *         description: Recovery action created
 */

/**
 * @swagger
 * /api/recovery-actions/client/{clientId}:
 *   get:
 *     summary: Get recovery actions by client
 *     tags: [Recovery Actions]
 *     parameters:
 *       - in: path
 *         name: clientId
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Recovery actions for client
 */

/**
 * @swagger
 * /api/recovery-actions/invoice/{invoiceId}:
 *   get:
 *     summary: Get recovery actions by invoice
 *     tags: [Recovery Actions]
 *     parameters:
 *       - in: path
 *         name: invoiceId
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Recovery actions for invoice
 */

/**
 * @swagger
 * /api/recovery-actions/{id}:
 *   put:
 *     summary: Update a recovery action
 *     tags: [Recovery Actions]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Recovery action updated
 *   delete:
 *     summary: Delete a recovery action
 *     tags: [Recovery Actions]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: Recovery action deleted
 */
```

#### `routes/users.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Users
 *   description: User management (admin)
 */

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users (admin only)
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: List of users
 */

/**
 * @swagger
 * /api/users/{id}:
 *   get:
 *     summary: Get user by ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: User data
 *   put:
 *     summary: Update user (admin only)
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               role:
 *                 type: string
 *                 enum: [agent, manager, admin]
 *     responses:
 *       200:
 *         description: User updated
 *   delete:
 *     summary: Delete user (admin only)
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: User deleted
 */
```

#### `routes/stats.js`:

```js
/**
 * @swagger
 * tags:
 *   name: Statistics
 *   description: Dashboard and analytics
 */

/**
 * @swagger
 * /api/stats/overview:
 *   get:
 *     summary: Get overview statistics
 *     tags: [Statistics]
 *     responses:
 *       200:
 *         description: Overview stats (totals, amounts, status breakdown)
 */

/**
 * @swagger
 * /api/stats/invoices:
 *   get:
 *     summary: Get invoice statistics
 *     tags: [Statistics]
 *     responses:
 *       200:
 *         description: Invoice stats (by status, overdue, monthly trends, payment methods)
 */

/**
 * @swagger
 * /api/stats/agents:
 *   get:
 *     summary: Get agent performance statistics
 *     tags: [Statistics]
 *     responses:
 *       200:
 *         description: Agent stats (actions count, amounts collected)
 */
```

## How to add these annotations

For each route file, paste the Swagger JSDoc comments **above** the route definitions (after the imports, before the `router.` calls). The `swagger-jsdoc` library will scan these comments and generate the OpenAPI spec.

## Verify

```bash
npm run dev

# Visit in browser:
open http://localhost:5000/api-docs

# Should show Swagger UI with all endpoints organized by tags:
# - Auth
# - Clients
# - Invoices
# - Payments
# - Recovery Actions
# - Users
# - Statistics
```

## Commit

```bash
git add .
git commit -m "Adem: add swagger documentation"
```
