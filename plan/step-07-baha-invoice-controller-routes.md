# Step 7 — Baha: Implement Invoice Controller and Routes

**Commit message:** `Baha: implement invoice controller and routes`

---

## What to do

Create full CRUD for invoices. An invoice belongs to a client, has an amount, due date, and a status that updates based on payments. Agents and managers can create/update. Managers and admins can delete.

## Files to create

### 1. `validators/invoiceValidator.js`

```js
const { check } = require('express-validator');

const createInvoiceValidator = [
  check('invoiceNumber')
    .notEmpty()
    .withMessage('Invoice number is required')
    .trim(),
  check('client')
    .notEmpty()
    .withMessage('Client ID is required')
    .isMongoId()
    .withMessage('Invalid client ID'),
  check('amount')
    .notEmpty()
    .withMessage('Amount is required')
    .isFloat({ min: 0 })
    .withMessage('Amount must be a positive number'),
  check('dueDate')
    .notEmpty()
    .withMessage('Due date is required')
    .isISO8601()
    .withMessage('Due date must be a valid date'),
];

const updateInvoiceValidator = [
  check('invoiceNumber')
    .optional()
    .trim(),
  check('client')
    .optional()
    .isMongoId()
    .withMessage('Invalid client ID'),
  check('amount')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Amount must be a positive number'),
  check('dueDate')
    .optional()
    .isISO8601()
    .withMessage('Due date must be a valid date'),
  check('status')
    .optional()
    .isIn(['unpaid', 'partially_paid', 'paid', 'overdue'])
    .withMessage('Status must be: unpaid, partially_paid, paid, or overdue'),
];

module.exports = { createInvoiceValidator, updateInvoiceValidator };
```

### 2. `controllers/invoiceController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const Invoice = require('../models/Invoice');
const Client = require('../models/Client');

// @desc    Get all invoices
// @route   GET /api/invoices
// @access  Private (all roles)
const getInvoices = asyncHandler(async (req, res) => {
  const invoices = await Invoice.find()
    .populate('client', 'name email phone')
    .populate('createdBy', 'name email');

  res.json(invoices);
});

// @desc    Get single invoice by ID
// @route   GET /api/invoices/:id
// @access  Private (all roles)
const getInvoiceById = asyncHandler(async (req, res) => {
  const invoice = await Invoice.findById(req.params.id)
    .populate('client', 'name email phone')
    .populate('createdBy', 'name email');

  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  res.json(invoice);
});

// @desc    Get invoices by client ID
// @route   GET /api/invoices/client/:clientId
// @access  Private (all roles)
const getInvoicesByClient = asyncHandler(async (req, res) => {
  const client = await Client.findById(req.params.clientId);

  if (!client) {
    res.status(404);
    throw new Error('Client not found');
  }

  const invoices = await Invoice.find({ client: req.params.clientId })
    .populate('client', 'name email phone')
    .populate('createdBy', 'name email');

  res.json(invoices);
});

// @desc    Create a new invoice
// @route   POST /api/invoices
// @access  Private (agent, manager)
const createInvoice = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { invoiceNumber, client, amount, dueDate } = req.body;

  // Check if client exists
  const clientExists = await Client.findById(client);
  if (!clientExists) {
    res.status(404);
    throw new Error('Client not found');
  }

  // Check for duplicate invoice number
  const invoiceExists = await Invoice.findOne({ invoiceNumber });
  if (invoiceExists) {
    res.status(400);
    throw new Error('Invoice number already exists');
  }

  const invoice = await Invoice.create({
    invoiceNumber,
    client,
    amount,
    dueDate,
    createdBy: req.user._id,
  });

  res.status(201).json(invoice);
});

// @desc    Update an invoice
// @route   PUT /api/invoices/:id
// @access  Private (agent, manager)
const updateInvoice = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const invoice = await Invoice.findById(req.params.id);

  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  const { invoiceNumber, client, amount, dueDate, status } = req.body;

  if (invoiceNumber) invoice.invoiceNumber = invoiceNumber;
  if (client) invoice.client = client;
  if (amount !== undefined) invoice.amount = amount;
  if (dueDate) invoice.dueDate = dueDate;
  if (status) invoice.status = status;

  const updatedInvoice = await invoice.save();
  res.json(updatedInvoice);
});

// @desc    Delete an invoice
// @route   DELETE /api/invoices/:id
// @access  Private (manager, admin)
const deleteInvoice = asyncHandler(async (req, res) => {
  const invoice = await Invoice.findById(req.params.id);

  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  await invoice.deleteOne();
  res.json({ message: 'Invoice deleted successfully' });
});

module.exports = {
  getInvoices,
  getInvoiceById,
  getInvoicesByClient,
  createInvoice,
  updateInvoice,
  deleteInvoice,
};
```

### 3. `routes/invoices.js`

```js
const express = require('express');
const router = express.Router();
const {
  getInvoices,
  getInvoiceById,
  getInvoicesByClient,
  createInvoice,
  updateInvoice,
  deleteInvoice,
} = require('../controllers/invoiceController');
const { protect, authorize } = require('../middleware/authMiddleware');
const { createInvoiceValidator, updateInvoiceValidator } = require('../validators/invoiceValidator');

router.use(protect); // All invoice routes require authentication

router
  .route('/')
  .get(getInvoices)
  .post(authorize('agent', 'manager'), createInvoiceValidator, createInvoice);

router.get('/client/:clientId', getInvoicesByClient);

router
  .route('/:id')
  .get(getInvoiceById)
  .put(authorize('agent', 'manager'), updateInvoiceValidator, updateInvoice)
  .delete(authorize('manager', 'admin'), deleteInvoice);

module.exports = router;
```

### 4. Update `app.js` — add invoice routes

Add this line in the routes section of `app.js`:

```js
const invoiceRoutes = require('./routes/invoices');
app.use('/api/invoices', invoiceRoutes);
```

## Folder structure after this step

```
├── controllers/
│   ├── authController.js
│   ├── clientController.js
│   └── invoiceController.js  ← NEW
├── routes/
│   ├── auth.js
│   ├── clients.js
│   └── invoices.js           ← NEW
├── validators/
│   ├── authValidator.js
│   ├── clientValidator.js
│   └── invoiceValidator.js   ← NEW
```

## Verify

```bash
npm run dev

# Create an invoice (must be logged in, need a valid client ID):
curl -X POST http://localhost:5000/api/invoices \
  -H "Content-Type: application/json" \
  -d '{"invoiceNumber":"INV-001","client":"<clientId>","amount":1500,"dueDate":"2026-03-15"}' \
  -b cookies.txt

# Get all invoices:
curl http://localhost:5000/api/invoices -b cookies.txt

# Get invoices for a specific client:
curl http://localhost:5000/api/invoices/client/<clientId> -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Baha: implement invoice controller and routes"
```
