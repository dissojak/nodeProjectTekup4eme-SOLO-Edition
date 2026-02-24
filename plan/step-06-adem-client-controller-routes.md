# Step 6 — Adem: Implement Client Controller and Routes

**Commit message:** `Adem: implement client controller and routes`

---

## What to do

Create full CRUD for managing clients (companies/individuals who owe debts). Only agents and managers can create/update clients. Managers and admins can delete.

## Files to create

### 1. `validators/clientValidator.js`

```js
const { check } = require('express-validator');

const createClientValidator = [
  check('name')
    .notEmpty()
    .withMessage('Client name is required')
    .isLength({ min: 2, max: 100 })
    .withMessage('Name must be between 2 and 100 characters'),
  check('email')
    .optional()
    .isEmail()
    .withMessage('Please provide a valid email'),
  check('phone')
    .optional()
    .isLength({ min: 8, max: 15 })
    .withMessage('Phone must be between 8 and 15 characters'),
  check('address')
    .optional()
    .isLength({ max: 200 })
    .withMessage('Address must be less than 200 characters'),
];

const updateClientValidator = [
  check('name')
    .optional()
    .isLength({ min: 2, max: 100 })
    .withMessage('Name must be between 2 and 100 characters'),
  check('email')
    .optional()
    .isEmail()
    .withMessage('Please provide a valid email'),
  check('phone')
    .optional()
    .isLength({ min: 8, max: 15 })
    .withMessage('Phone must be between 8 and 15 characters'),
  check('address')
    .optional()
    .isLength({ max: 200 })
    .withMessage('Address must be less than 200 characters'),
];

module.exports = { createClientValidator, updateClientValidator };
```

### 2. `controllers/clientController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const Client = require('../models/Client');

// @desc    Get all clients
// @route   GET /api/clients
// @access  Private (all roles)
const getClients = asyncHandler(async (req, res) => {
  const clients = await Client.find().populate('createdBy', 'name email');
  res.json(clients);
});

// @desc    Get single client by ID
// @route   GET /api/clients/:id
// @access  Private (all roles)
const getClientById = asyncHandler(async (req, res) => {
  const client = await Client.findById(req.params.id).populate('createdBy', 'name email');

  if (!client) {
    res.status(404);
    throw new Error('Client not found');
  }

  res.json(client);
});

// @desc    Create a new client
// @route   POST /api/clients
// @access  Private (agent, manager)
const createClient = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { name, email, phone, address } = req.body;

  const client = await Client.create({
    name,
    email,
    phone,
    address,
    createdBy: req.user._id,
  });

  res.status(201).json(client);
});

// @desc    Update a client
// @route   PUT /api/clients/:id
// @access  Private (agent, manager)
const updateClient = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const client = await Client.findById(req.params.id);

  if (!client) {
    res.status(404);
    throw new Error('Client not found');
  }

  const { name, email, phone, address } = req.body;

  client.name = name || client.name;
  client.email = email || client.email;
  client.phone = phone || client.phone;
  client.address = address || client.address;

  const updatedClient = await client.save();
  res.json(updatedClient);
});

// @desc    Delete a client
// @route   DELETE /api/clients/:id
// @access  Private (manager, admin)
const deleteClient = asyncHandler(async (req, res) => {
  const client = await Client.findById(req.params.id);

  if (!client) {
    res.status(404);
    throw new Error('Client not found');
  }

  await client.deleteOne();
  res.json({ message: 'Client deleted successfully' });
});

module.exports = { getClients, getClientById, createClient, updateClient, deleteClient };
```

### 3. `routes/clients.js`

```js
const express = require('express');
const router = express.Router();
const {
  getClients,
  getClientById,
  createClient,
  updateClient,
  deleteClient,
} = require('../controllers/clientController');
const { protect, authorize } = require('../middleware/authMiddleware');
const { createClientValidator, updateClientValidator } = require('../validators/clientValidator');

router.use(protect); // All client routes require authentication

router
  .route('/')
  .get(getClients)
  .post(authorize('agent', 'manager'), createClientValidator, createClient);

router
  .route('/:id')
  .get(getClientById)
  .put(authorize('agent', 'manager'), updateClientValidator, updateClient)
  .delete(authorize('manager', 'admin'), deleteClient);

module.exports = router;
```

### 4. Update `app.js` — add client routes

Add this line in the routes section of `app.js`:

```js
const clientRoutes = require('./routes/clients');
app.use('/api/clients', clientRoutes);
```

## Folder structure after this step

```
├── controllers/
│   ├── authController.js    (from step 4)
│   └── clientController.js  ← NEW
├── routes/
│   ├── auth.js              (from step 4)
│   └── clients.js           ← NEW
├── validators/
│   ├── authValidator.js     (from step 4)
│   └── clientValidator.js   ← NEW
```

## Verify

```bash
npm run dev

# Login first to get a cookie:
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"adem@test.com","password":"123456"}' \
  -c cookies.txt

# Create a client:
curl -X POST http://localhost:5000/api/clients \
  -H "Content-Type: application/json" \
  -d '{"name":"Company ABC","email":"abc@company.com","phone":"12345678"}' \
  -b cookies.txt

# Get all clients:
curl http://localhost:5000/api/clients -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Adem: implement client controller and routes"
```
