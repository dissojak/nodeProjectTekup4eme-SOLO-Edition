# Step 8 — Adem: Implement Payment Controller and Routes

**Commit message:** `Adem: implement payment controller and routes`

---

## What to do

Create payment recording — when a client pays against an invoice, record it and auto-update invoice status (unpaid → partially_paid → paid). Uses the **Strategy pattern** for payment methods.

## Design pattern: Strategy

The Strategy pattern lets us handle different payment methods (cash, check, transfer) with different processing logic, without using if/else chains in the controller. Each strategy is swappable.

## Files to create

### 1. `strategies/paymentStrategies.js`

```js
// Strategy pattern — each payment method has its own processing logic
// In a real app, these could connect to different external services

const paymentStrategies = {
  cash: {
    process: (paymentData) => {
      // Cash payment — immediate processing
      return {
        ...paymentData,
        processedAt: new Date(),
        confirmation: `CASH-${Date.now()}`,
      };
    },
  },

  check: {
    process: (paymentData) => {
      // Check payment — may need clearing time
      return {
        ...paymentData,
        processedAt: new Date(),
        confirmation: `CHK-${Date.now()}`,
        note: paymentData.note
          ? `${paymentData.note} (Check payment)`
          : 'Check payment - allow clearing time',
      };
    },
  },

  transfer: {
    process: (paymentData) => {
      // Bank transfer — reference number tracking
      return {
        ...paymentData,
        processedAt: new Date(),
        confirmation: `TRF-${Date.now()}`,
      };
    },
  },
};

const getPaymentStrategy = (method) => {
  const strategy = paymentStrategies[method];
  if (!strategy) {
    throw new Error(`Unknown payment method: ${method}`);
  }
  return strategy;
};

module.exports = { getPaymentStrategy };
```

### 2. `validators/paymentValidator.js`

```js
const { check } = require('express-validator');

const createPaymentValidator = [
  check('invoice')
    .notEmpty()
    .withMessage('Invoice ID is required')
    .isMongoId()
    .withMessage('Invalid invoice ID'),
  check('amount')
    .notEmpty()
    .withMessage('Payment amount is required')
    .isFloat({ min: 0.01 })
    .withMessage('Payment amount must be greater than 0'),
  check('paymentMethod')
    .notEmpty()
    .withMessage('Payment method is required')
    .isIn(['cash', 'check', 'transfer'])
    .withMessage('Payment method must be: cash, check, or transfer'),
  check('paymentDate')
    .optional()
    .isISO8601()
    .withMessage('Payment date must be a valid date'),
  check('note')
    .optional()
    .isLength({ max: 500 })
    .withMessage('Note must be less than 500 characters'),
];

module.exports = { createPaymentValidator };
```

### 3. `controllers/paymentController.js`

```js
const asyncHandler = require('express-async-handler');
const { validationResult } = require('express-validator');
const Payment = require('../models/Payment');
const Invoice = require('../models/Invoice');
const { getPaymentStrategy } = require('../strategies/paymentStrategies');

// @desc    Get all payments
// @route   GET /api/payments
// @access  Private (all roles)
const getPayments = asyncHandler(async (req, res) => {
  const payments = await Payment.find()
    .populate('invoice', 'invoiceNumber amount status')
    .populate('recordedBy', 'name email');

  res.json(payments);
});

// @desc    Get payments by invoice ID
// @route   GET /api/payments/invoice/:invoiceId
// @access  Private (all roles)
const getPaymentsByInvoice = asyncHandler(async (req, res) => {
  const invoice = await Invoice.findById(req.params.invoiceId);

  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  const payments = await Payment.find({ invoice: req.params.invoiceId })
    .populate('invoice', 'invoiceNumber amount status')
    .populate('recordedBy', 'name email');

  res.json(payments);
});

// @desc    Record a new payment
// @route   POST /api/payments
// @access  Private (agent, manager)
const createPayment = asyncHandler(async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400);
    throw new Error(errors.array().map((e) => e.msg).join(', '));
  }

  const { invoice: invoiceId, amount, paymentMethod, paymentDate, note } = req.body;

  // Find the invoice
  const invoice = await Invoice.findById(invoiceId);
  if (!invoice) {
    res.status(404);
    throw new Error('Invoice not found');
  }

  // Check if invoice is already fully paid
  if (invoice.status === 'paid') {
    res.status(400);
    throw new Error('This invoice is already fully paid');
  }

  // Check if payment exceeds remaining balance
  const remainingBalance = invoice.amount - invoice.amountPaid;
  if (amount > remainingBalance) {
    res.status(400);
    throw new Error(
      `Payment amount (${amount}) exceeds remaining balance (${remainingBalance})`
    );
  }

  // Use Strategy pattern to process payment
  const strategy = getPaymentStrategy(paymentMethod);
  const processedPayment = strategy.process({
    amount,
    paymentMethod,
    note,
  });

  // Create payment record
  const payment = await Payment.create({
    invoice: invoiceId,
    amount: processedPayment.amount,
    paymentMethod: processedPayment.paymentMethod,
    paymentDate: paymentDate || Date.now(),
    note: processedPayment.note || note,
    recordedBy: req.user._id,
  });

  // Update invoice: add payment to amountPaid and update status
  invoice.amountPaid += amount;

  if (invoice.amountPaid >= invoice.amount) {
    invoice.status = 'paid';
  } else if (invoice.amountPaid > 0) {
    invoice.status = 'partially_paid';
  }

  await invoice.save();

  res.status(201).json({
    payment,
    invoiceStatus: invoice.status,
    remainingBalance: invoice.amount - invoice.amountPaid,
    confirmation: processedPayment.confirmation,
  });
});

module.exports = { getPayments, getPaymentsByInvoice, createPayment };
```

### 4. `routes/payments.js`

```js
const express = require('express');
const router = express.Router();
const {
  getPayments,
  getPaymentsByInvoice,
  createPayment,
} = require('../controllers/paymentController');
const { protect, authorize } = require('../middleware/authMiddleware');
const { createPaymentValidator } = require('../validators/paymentValidator');

router.use(protect); // All payment routes require authentication

router
  .route('/')
  .get(getPayments)
  .post(authorize('agent', 'manager'), createPaymentValidator, createPayment);

router.get('/invoice/:invoiceId', getPaymentsByInvoice);

module.exports = router;
```

### 5. Update `app.js` — add payment routes

Add this line in the routes section of `app.js`:

```js
const paymentRoutes = require('./routes/payments');
app.use('/api/payments', paymentRoutes);
```

## How the Strategy pattern works here

```
Controller receives paymentMethod: "check"
       ↓
getPaymentStrategy("check")  →  selects the check strategy
       ↓
strategy.process(data)  →  adds check-specific note + confirmation code
       ↓
Payment saved with processed data
```

If tomorrow you add a new payment method (e.g., "mobile_money"), you just add a new strategy object — no controller changes needed.

## Folder structure after this step

```
├── controllers/
│   ├── authController.js
│   ├── clientController.js
│   └── paymentController.js   ← NEW
├── routes/
│   ├── auth.js
│   ├── clients.js
│   └── payments.js            ← NEW
├── validators/
│   ├── authValidator.js
│   ├── clientValidator.js
│   └── paymentValidator.js    ← NEW
├── strategies/
│   └── paymentStrategies.js   ← NEW
```

## Verify

```bash
npm run dev

# Record a payment:
curl -X POST http://localhost:5000/api/payments \
  -H "Content-Type: application/json" \
  -d '{"invoice":"<invoiceId>","amount":500,"paymentMethod":"cash","note":"First payment"}' \
  -b cookies.txt

# Response should include:
# - payment object
# - updated invoiceStatus (partially_paid or paid)
# - remainingBalance
# - confirmation code (e.g., CASH-1708...)

# Get all payments:
curl http://localhost:5000/api/payments -b cookies.txt

# Get payments for a specific invoice:
curl http://localhost:5000/api/payments/invoice/<invoiceId> -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Adem: implement payment controller and routes"
```
