# Architecture & Conventions

## Project structure

```
├── config/          → Database connection (Singleton pattern)
├── controllers/     → Route handlers (one file per resource)
├── middleware/       → Auth middleware, error handler, validators
├── models/          → Mongoose schemas (User, Client, Invoice, Payment, RecoveryAction)
├── routes/          → Express routers (one file per resource)
├── validators/      → express-validator chains (one file per resource)
├── utils/           → Helpers (HttpError, generateToken)
├── strategies/      → Strategy pattern implementations (if needed)
├── tests/           → Jest + Supertest unit tests
├── plan/            → Step-by-step plan files (DO NOT modify)
├── app.js           → Express app setup (middleware chain)
├── server.js        → Server entry point (separated for testability)
```

## Design patterns used

1. **MVC** — Models in `models/`, Controllers in `controllers/`, Routes as the view layer
2. **Singleton** — `config/db.js` creates one shared DB connection
3. **Chain of Responsibility** — Middleware chain (helmet → cors → morgan → json → cookies → auth → routes)
4. **Decorator** — `asyncHandler` wraps controllers to catch async errors
5. **Factory** — `HttpError` creates error objects with status codes
6. **Strategy** — For recovery action types or payment methods (implemented in strategies/ if needed)

## Key conventions

### Adding a new resource (controller + routes)

Follow this pattern (already done for auth, replicate for client/invoice/payment/recovery/user):

1. Create `validators/<resource>Validator.js` with validation chains
2. Create `controllers/<resource>Controller.js` with CRUD handlers
3. Create `routes/<resource>.js` wiring validators → controllers
4. Register routes in `app.js` under the routes section:
   ```js
   const resourceRoutes = require('./routes/resource');
   app.use('/api/resource', resourceRoutes);
   ```

### Route registration in app.js

All routes go in the `// ============ ALL ROUTES WILL BE ADDED HERE ============` section.
Add one `require` + one `app.use` per resource. Keep them together, no extra comments.

Example of what the routes section should look like when done:
```js
const authRoutes = require('./routes/auth');
const clientRoutes = require('./routes/client');
const invoiceRoutes = require('./routes/invoice');
const paymentRoutes = require('./routes/payment');
const recoveryRoutes = require('./routes/recovery');
const userRoutes = require('./routes/user');
const statsRoutes = require('./routes/stats');

app.use('/api/auth', authRoutes);
app.use('/api/clients', clientRoutes);
app.use('/api/invoices', invoiceRoutes);
app.use('/api/payments', paymentRoutes);
app.use('/api/recovery', recoveryRoutes);
app.use('/api/users', userRoutes);
app.use('/api/stats', statsRoutes);
```

### Authentication

- JWT stored in httpOnly cookie (not Authorization header)
- `protect` middleware verifies token and attaches `req.user`
- `authorize('admin', 'manager')` restricts by role
- Password field has `select: false` — use `.select('+password')` when needed for login

### API response format

- Success: `res.status(200).json(data)` or `res.status(201).json(data)`
- Error: thrown via `HttpError` and caught by error middleware
- Validation errors: `res.status(400).json({ errors: errors.array() })`

### Environment variables

```
PORT=5500
MONGO_URI=<mongodb connection string>
JWT_SECRET=<long random string>
JWT_EXPIRE=30d
NODE_ENV=development
```

The `.env` file is already configured. Do not change it unless adding new variables.
