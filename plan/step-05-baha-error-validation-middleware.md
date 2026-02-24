# Step 5 — Baha: Add Error and Validation Middleware

**Commit message:** `Baha: add error and validation middleware`

---

## What to do

Create the global error handling middleware and a reusable validation middleware runner. These are used by ALL routes going forward.

## Design patterns

- **Chain of Responsibility** — error middleware is the last link in the chain, catches everything
- **Decorator** — `validate` wraps express-validator results into a clean error response

## Files to create

### 1. `middleware/errorMiddleware.js`

Two functions:
- `notFound` — catches requests to undefined routes (404)
- `errorHandler` — global error handler (must be registered LAST in app.js)

```js
// Handle 404 — route not found
const notFound = (req, res, next) => {
  const error = new Error(`Not Found - ${req.originalUrl}`);
  res.status(404);
  next(error);
};

// Global error handler
const errorHandler = (err, req, res, next) => {
  // If headers already sent, delegate to Express default handler
  if (res.headersSent) {
    return next(err);
  }

  let statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  let message = err.message;

  // Mongoose bad ObjectId — cast error
  if (err.name === 'CastError' && err.kind === 'ObjectId') {
    statusCode = 404;
    message = 'Resource not found';
  }

  // Mongoose duplicate key error
  if (err.code === 11000) {
    statusCode = 400;
    const field = Object.keys(err.keyValue).join(', ');
    message = `Duplicate value for field: ${field}`;
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    statusCode = 400;
    message = Object.values(err.errors)
      .map((val) => val.message)
      .join(', ');
  }

  res.status(statusCode).json({
    message,
    stack: process.env.NODE_ENV === 'production' ? null : err.stack,
  });
};

module.exports = { notFound, errorHandler };
```

### 2. `middleware/validateMiddleware.js`

A reusable middleware that runs express-validator checks and returns a clean 400 error if validation fails. This can be used instead of checking `validationResult` inside every controller.

```js
const { validationResult } = require('express-validator');

const validate = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(
      errors
        .array()
        .map((e) => e.msg)
        .join(', ')
    );
  }

  next();
};

module.exports = validate;
```

### 3. Update `app.js` — wire error middleware

Add these lines at the **very bottom** of `app.js`, after all routes:

```js
// Error handling middleware (must be LAST)
const { notFound, errorHandler } = require('./middleware/errorMiddleware');
app.use(notFound);
app.use(errorHandler);
```

> **Note:** Unlike Artifex (which defined this middleware but never used it), we actually wire it into the app.

## How `validate` middleware works in routes (preview)

From now on, routes can use it like this:

```js
const validate = require('../middleware/validateMiddleware');
const { registerValidator } = require('../validators/authValidator');

// Validation chain → validate middleware → controller
router.post('/register', registerValidator, validate, register);
```

> **Optional improvement:** You can go back and update `routes/auth.js` to use `validate` middleware instead of checking `validationResult` inside the auth controller. This is cleaner but not required — both approaches work.

## Folder structure after this step

```
├── middleware/
│   ├── authMiddleware.js    (from step 3)
│   ├── errorMiddleware.js   ← NEW
│   └── validateMiddleware.js ← NEW
```

## Verify

```bash
npm run dev

# Test 404 handler:
curl http://localhost:5000/api/nonexistent
# Should return: {"message":"Not Found - /api/nonexistent","stack":"..."}

# Test validation error:
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{}'
# Should return: 400 with validation error messages
```

## Commit

```bash
git add .
git commit -m "Baha: add error and validation middleware"
```
