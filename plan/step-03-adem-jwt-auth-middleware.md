# Step 3 — Adem: Add JWT Auth and Middleware

**Commit message:** `Adem: add JWT auth and middleware`

---

## What to do

Create the JWT token generation utility and the auth protection middleware (protect + authorize by role).

## Design patterns

- **Chain of Responsibility** — middleware checks auth, then passes to next handler
- **Decorator** — `asyncHandler` wraps functions to catch async errors

## Files to create

### 1. `utils/generateToken.js`

Generates a JWT and stores it in an httpOnly cookie.

```js
const jwt = require('jsonwebtoken');

const generateToken = (res, userId) => {
  const token = jwt.sign({ userId }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || '30d',
  });

  res.cookie('jwt', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    sameSite: 'strict',
    maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
  });

  return token;
};

module.exports = generateToken;
```

### 2. `middleware/authMiddleware.js`

Two middleware functions:
- `protect` — verifies JWT token, attaches `req.user`
- `authorize(...roles)` — checks if user has one of the allowed roles

```js
const jwt = require('jsonwebtoken');
const asyncHandler = require('express-async-handler');
const User = require('../models/User');

// Protect routes — must be logged in
const protect = asyncHandler(async (req, res, next) => {
  let token = req.cookies.jwt;

  if (!token) {
    res.status(401);
    throw new Error('Not authorized, no token');
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = await User.findById(decoded.userId).select('-password');

    if (!req.user) {
      res.status(401);
      throw new Error('Not authorized, user not found');
    }

    next();
  } catch (error) {
    res.status(401);
    throw new Error('Not authorized, token invalid');
  }
});

// Authorize by role — must have one of the specified roles
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      res.status(401);
      throw new Error('Not authorized');
    }

    if (!roles.includes(req.user.role)) {
      res.status(403);
      throw new Error(`Role '${req.user.role}' is not authorized to access this resource`);
    }

    next();
  };
};

module.exports = { protect, authorize };
```

## How it works in routes (preview — actual routes come later)

```js
// Example usage:
router.get('/users', protect, authorize('admin'), getUsers);
//                    ↑ must be logged in
//                              ↑ must be admin
```

## Folder structure after this step

```
├── middleware/
│   └── authMiddleware.js
├── utils/
│   ├── HttpError.js
│   └── generateToken.js
```

## Verify

```bash
npm run dev
# Server should still start without errors
```

## Commit

```bash
git add .
git commit -m "Adem: add JWT auth and middleware"
```
