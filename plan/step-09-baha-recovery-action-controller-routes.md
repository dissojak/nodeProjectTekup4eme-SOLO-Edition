# Step 9 — Baha: Implement Recovery Action Controller and Routes

**Commit message:** `Baha: implement recovery action controller and routes`

---

## What to do

Create CRUD for recovery/collection actions. These track what actions were taken to recover a debt (phone calls, emails, letters, visits, legal action). Linked to both an invoice and a client.

## Files to create

### 1. `validators/recoveryActionValidator.js`

```js
const { check } = require('express-validator');

const createRecoveryActionValidator = [
  check('invoice')
    .notEmpty()
    .withMessage('Invoice ID is required')
    .isMongoId()
    .withMessage('Invalid invoice ID'),
  check('client')
    .notEmpty()
    .withMessage('Client ID is required')
    .isMongoId()
    .withMessage('Invalid client ID'),
  check('actionType')
    .notEmpty()
    .withMessage('Action type is required')
    .isIn(['phone_call', 'email', 'letter', 'visit', 'legal'])
    .withMessage('Action type must be: phone_call, email, letter, visit, or legal'),
  check('note')
    .optional()
    .isLength({ max: 1000 })
    .withMessage('Note must be less than 1000 characters'),
  check('result')
    .optional()
    .isLength({ max: 500 })
    .withMessage('Result must be less than 500 characters'),
  check('actionDate')
    .optional()
    .isISO8601()
    .withMessage('Action date must be a valid date'),
];

const updateRecoveryActionValidator = [
  check('actionType')
    .optional()
    .isIn(['phone_call', 'email', 'letter', 'visit', 'legal'])
    .withMessage('Action type must be: phone_call, email, letter, visit, or legal'),
  check('note')
    .optional()
    .isLength({ max: 1000 })
    .withMessage('Note must be less than 1000 characters'),
  check('result')
    .optional()
    .isLength({ max: 500 })
    .withMessage('Result must be less than 500 characters'),
  check('actionDate')
    .optional()
    .isISO8601()
    .withMessage('Action date must be a valid date'),
];

module.exports = { createRecoveryActionValidator, updateRecoveryActionValidator };
```

### 2. `controllers/recoveryActionController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const RecoveryAction = require('../models/RecoveryAction');
const Invoice = require('../models/Invoice');
const Client = require('../models/Client');

// @desc    Get all recovery actions
// @route   GET /api/recovery-actions
// @access  Private (all roles)
const getRecoveryActions = asyncHandler(async (req, res) => {
  const actions = await RecoveryAction.find()
    .populate('invoice', 'invoiceNumber amount status')
    .populate('client', 'name email phone')
    .populate('performedBy', 'name email');

  res.json(actions);
});

// @desc    Get recovery actions by client ID
// @route   GET /api/recovery-actions/client/:clientId
// @access  Private (all roles)
const getRecoveryActionsByClient = asyncHandler(async (req, res) => {
  const client = await Client.findById(req.params.clientId);

  if (!client) {
    res.status(404);
    throw new Error('Client not found');
  }

  const actions = await RecoveryAction.find({ client: req.params.clientId })
    .populate('invoice', 'invoiceNumber amount status')
    .populate('client', 'name email phone')
    .populate('performedBy', 'name email');

  res.json(actions);
});

// @desc    Get recovery actions by invoice ID
// @route   GET /api/recovery-actions/invoice/:invoiceId
// @access  Private (all roles)
const getRecoveryActionsByInvoice = asyncHandler(async (req, res) => {
  const invoice = await Invoice.findById(req.params.invoiceId);

  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  const actions = await RecoveryAction.find({ invoice: req.params.invoiceId })
    .populate('invoice', 'invoiceNumber amount status')
    .populate('client', 'name email phone')
    .populate('performedBy', 'name email');

  res.json(actions);
});

// @desc    Create a new recovery action
// @route   POST /api/recovery-actions
// @access  Private (agent, manager)
const createRecoveryAction = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { invoice, client, actionType, note, result, actionDate } = req.body;

  // Verify invoice exists
  const invoiceExists = await Invoice.findById(invoice);
  if (!invoiceExists) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  // Verify client exists
  const clientExists = await Client.findById(client);
  if (!clientExists) {
    res.status(404);
    throw new Error('Client not found');
  }

  const action = await RecoveryAction.create({
    invoice,
    client,
    actionType,
    note,
    result,
    actionDate: actionDate || Date.now(),
    performedBy: req.user._id,
  });

  res.status(201).json(action);
});

// @desc    Update a recovery action
// @route   PUT /api/recovery-actions/:id
// @access  Private (agent, manager)
const updateRecoveryAction = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const action = await RecoveryAction.findById(req.params.id);

  if (!action) {
    res.status(404);
    throw new Error('Recovery action not found');
  }

  const { actionType, note, result, actionDate } = req.body;

  if (actionType) action.actionType = actionType;
  if (note !== undefined) action.note = note;
  if (result !== undefined) action.result = result;
  if (actionDate) action.actionDate = actionDate;

  const updatedAction = await action.save();
  res.json(updatedAction);
});

// @desc    Delete a recovery action
// @route   DELETE /api/recovery-actions/:id
// @access  Private (manager, admin)
const deleteRecoveryAction = asyncHandler(async (req, res) => {
  const action = await RecoveryAction.findById(req.params.id);

  if (!action) {
    res.status(404);
    throw new Error('Recovery action not found');
  }

  await action.deleteOne();
  res.json({ message: 'Recovery action deleted successfully' });
});

module.exports = {
  getRecoveryActions,
  getRecoveryActionsByClient,
  getRecoveryActionsByInvoice,
  createRecoveryAction,
  updateRecoveryAction,
  deleteRecoveryAction,
};
```

### 3. `routes/recoveryActions.js`

```js
const express = require('express');
const router = express.Router();
const {
  getRecoveryActions,
  getRecoveryActionsByClient,
  getRecoveryActionsByInvoice,
  createRecoveryAction,
  updateRecoveryAction,
  deleteRecoveryAction,
} = require('../controllers/recoveryActionController');
const { protect, authorize } = require('../middleware/authMiddleware');
const {
  createRecoveryActionValidator,
  updateRecoveryActionValidator,
} = require('../validators/recoveryActionValidator');

router.use(protect); // All recovery action routes require authentication

router
  .route('/')
  .get(getRecoveryActions)
  .post(authorize('agent', 'manager'), createRecoveryActionValidator, createRecoveryAction);

router.get('/client/:clientId', getRecoveryActionsByClient);
router.get('/invoice/:invoiceId', getRecoveryActionsByInvoice);

router
  .route('/:id')
  .put(authorize('agent', 'manager'), updateRecoveryActionValidator, updateRecoveryAction)
  .delete(authorize('manager', 'admin'), deleteRecoveryAction);

module.exports = router;
```

### 4. Update `app.js` — add recovery action routes

Add this line in the routes section of `app.js`:

```js
const recoveryActionRoutes = require('./routes/recoveryActions');
app.use('/api/recovery-actions', recoveryActionRoutes);
```

## Folder structure after this step

```
├── controllers/
│   ├── authController.js
│   ├── clientController.js
│   ├── paymentController.js
│   └── recoveryActionController.js  ← NEW
├── routes/
│   ├── auth.js
│   ├── clients.js
│   ├── payments.js
│   └── recoveryActions.js           ← NEW
├── validators/
│   ├── authValidator.js
│   ├── clientValidator.js
│   ├── paymentValidator.js
│   └── recoveryActionValidator.js   ← NEW
```

## Verify

```bash
npm run dev

# Create a recovery action:
curl -X POST http://localhost:5000/api/recovery-actions \
  -H "Content-Type: application/json" \
  -d '{"invoice":"<invoiceId>","client":"<clientId>","actionType":"phone_call","note":"Called client about overdue invoice","result":"Client promised to pay next week"}' \
  -b cookies.txt

# Get all recovery actions:
curl http://localhost:5000/api/recovery-actions -b cookies.txt

# Get by client:
curl http://localhost:5000/api/recovery-actions/client/<clientId> -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Baha: implement recovery action controller and routes"
```
