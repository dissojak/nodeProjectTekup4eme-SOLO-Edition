# Step 10 — Baha: Implement User Management Controller and Routes

**Commit message:** `Baha: implement user management controller and routes`

---

## What to do

Create user management endpoints for admins — list all users, get a user, update user (change role, etc.), and delete user. Different from auth (which handles register/login/logout).

## Files to create

### 1. `controllers/userController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const User = require('../models/User');

// @desc    Get all users
// @route   GET /api/users
// @access  Private (admin only)
const getUsers = asyncHandler(async (req, res) => {
  const users = await User.find().select('-password');
  res.json(users);
});

// @desc    Get user by ID
// @route   GET /api/users/:id
// @access  Private (admin, manager)
const getUserById = asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id).select('-password');

  if (!user) {
    res.status(404);
    throw new Error('User not found');
  }

  res.json(user);
});

// @desc    Update user (admin can change roles, etc.)
// @route   PUT /api/users/:id
// @access  Private (admin only)
const updateUser = asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    res.status(404);
    throw new Error('User not found');
  }

  const { name, email, role } = req.body;

  if (name) user.name = name;
  if (email) user.email = email;
  if (role) user.role = role;

  const updatedUser = await user.save();

  res.json({
    _id: updatedUser._id,
    name: updatedUser.name,
    email: updatedUser.email,
    role: updatedUser.role,
  });
});

// @desc    Delete user
// @route   DELETE /api/users/:id
// @access  Private (admin only)
const deleteUser = asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    res.status(404);
    throw new Error('User not found');
  }

  // Prevent admin from deleting themselves
  if (user._id.toString() === req.user._id.toString()) {
    res.status(400);
    throw new Error('Cannot delete your own account');
  }

  await user.deleteOne();
  res.json({ message: 'User deleted successfully' });
});

module.exports = { getUsers, getUserById, updateUser, deleteUser };
```

### 2. `routes/users.js`

```js
const express = require('express');
const router = express.Router();
const {
  getUsers,
  getUserById,
  updateUser,
  deleteUser,
} = require('../controllers/userController');
const { protect, authorize } = require('../middleware/authMiddleware');

router.use(protect); // All user management routes require authentication

router.route('/').get(authorize('admin'), getUsers);

router
  .route('/:id')
  .get(authorize('admin', 'manager'), getUserById)
  .put(authorize('admin'), updateUser)
  .delete(authorize('admin'), deleteUser);

module.exports = router;
```

### 3. Update `app.js` — add user routes

Add this line in the routes section of `app.js`:

```js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

## Folder structure after this step

```
├── controllers/
│   ├── authController.js
│   ├── clientController.js
│   ├── invoiceController.js
│   ├── paymentController.js
│   ├── recoveryActionController.js
│   └── userController.js          ← NEW
├── routes/
│   ├── auth.js
│   ├── clients.js
│   ├── invoices.js
│   ├── payments.js
│   ├── recoveryActions.js
│   └── users.js                   ← NEW
```

## Verify

```bash
npm run dev

# Must be logged in as admin:

# Get all users:
curl http://localhost:5000/api/users -b cookies.txt

# Get a specific user:
curl http://localhost:5000/api/users/<userId> -b cookies.txt

# Update a user's role:
curl -X PUT http://localhost:5000/api/users/<userId> \
  -H "Content-Type: application/json" \
  -d '{"role":"manager"}' \
  -b cookies.txt

# Delete a user:
curl -X DELETE http://localhost:5000/api/users/<userId> -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Baha: implement user management controller and routes"
```
