# Step 4 — Adem: Implement Auth Controller and Routes

**Commit message:** `Adem: implement auth controller and routes`

---

## What to do

Create the authentication system: register, login, logout, and get current user profile. Wire the auth routes into `app.js`.

## Files to create

### 1. `validators/authValidator.js`

**Design pattern:** Chain of Responsibility — each `check()` is a link in the validation chain.

```js
const { check } = require('express-validator');

const registerValidator = [
  check('name')
    .notEmpty()
    .withMessage('Name is required')
    .isLength({ min: 3, max: 50 })
    .withMessage('Name must be between 3 and 50 characters'),
  check('email')
    .notEmpty()
    .withMessage('Email is required')
    .isEmail()
    .withMessage('Please provide a valid email'),
  check('password')
    .notEmpty()
    .withMessage('Password is required')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters'),
  check('role')
    .optional()
    .isIn(['agent', 'manager', 'admin'])
    .withMessage('Role must be agent, manager, or admin'),
];

const loginValidator = [
  check('email')
    .notEmpty()
    .withMessage('Email is required')
    .isEmail()
    .withMessage('Please provide a valid email'),
  check('password')
    .notEmpty()
    .withMessage('Password is required'),
];

module.exports = { registerValidator, loginValidator };
```

### 2. `controllers/authController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const User = require('../models/User');
const generateToken = require('../utils/generateToken');

// @desc    Register a new user
// @route   POST /api/auth/register
// @access  Public
const register = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { name, email, password, role } = req.body;

  // Check if user already exists
  const userExists = await User.findOne({ email });
  if (userExists) {
    res.status(400);
    throw new Error('User already exists with this email');
  }

  const user = await User.create({ name, email, password, role });

  generateToken(res, user._id);

  res.status(201).json({
    _id: user._id,
    name: user.name,
    email: user.email,
    role: user.role,
  });
});

// @desc    Login user
// @route   POST /api/auth/login
// @access  Public
const login = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { email, password } = req.body;

  // Find user and explicitly include password field
  const user = await User.findOne({ email }).select('+password');

  if (!user || !(await user.matchPassword(password))) {
    res.status(401);
    throw new Error('Invalid email or password');
  }

  generateToken(res, user._id);

  res.json({
    _id: user._id,
    name: user.name,
    email: user.email,
    role: user.role,
  });
});

// @desc    Logout user (clear cookie)
// @route   POST /api/auth/logout
// @access  Private
const logout = asyncHandler(async (req, res) => {
  res.cookie('jwt', '', {
    httpOnly: true,
    expires: new Date(0),
  });

  res.json({ message: 'Logged out successfully' });
});

// @desc    Get current user profile
// @route   GET /api/auth/me
// @access  Private
const getMe = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  res.json({
    _id: user._id,
    name: user.name,
    email: user.email,
    role: user.role,
  });
});

module.exports = { register, login, logout, getMe };
```

### 3. `routes/auth.js`

```js
const express = require('express');
const router = express.Router();
const { register, login, logout, getMe } = require('../controllers/authController');
const { protect } = require('../middleware/authMiddleware');
const { registerValidator, loginValidator } = require('../validators/authValidator');

router.post('/register', registerValidator, register);
router.post('/login', loginValidator, login);
router.post('/logout', protect, logout);
router.get('/me', protect, getMe);

module.exports = router;
```

### 4. Update `app.js` — add auth routes

Add this line after the middleware section, before the root route:

```js
// Routes
const authRoutes = require('./routes/auth');
app.use('/api/auth', authRoutes);
```

## Folder structure after this step

```
├── controllers/
│   └── authController.js
├── routes/
│   └── auth.js
├── validators/
│   └── authValidator.js
```

## Verify

```bash
npm run dev

# Test with curl or Postman:

# Register
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Adem","email":"adem@test.com","password":"123456","role":"admin"}'

# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"adem@test.com","password":"123456"}'

# Get profile (use the cookie from login)
curl http://localhost:5000/api/auth/me --cookie "jwt=<token>"
```

## Commit

```bash
git add .
git commit -m "Adem: implement auth controller and routes"
```
