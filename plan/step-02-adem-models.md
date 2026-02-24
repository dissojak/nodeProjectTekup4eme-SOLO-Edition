# Step 2 — Adem: Add All Models

**Commit message:** `Adem: add all models`

---

## What to do

Create all 5 Mongoose models and the custom HttpError utility class.

## Files to create

### 1. `utils/HttpError.js`

**Design pattern:** Factory — creates custom error objects with HTTP status codes.

```js
class HttpError extends Error {
  constructor(message, errorCode) {
    super(message);
    this.code = errorCode;
  }
}

module.exports = HttpError;
```

### 2. `models/User.js`

Roles: `agent`, `manager`, `admin`. Password hashed with bcrypt on save.

```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
      minlength: 3,
      maxlength: 50,
    },
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      trim: true,
      lowercase: true,
    },
    password: {
      type: String,
      required: [true, 'Password is required'],
      minlength: 6,
      select: false, // Don't return password in queries by default
    },
    role: {
      type: String,
      enum: ['agent', 'manager', 'admin'],
      default: 'agent',
    },
  },
  {
    timestamps: true, // Adds createdAt and updatedAt automatically
  }
);

// Hash password before saving (Decorator-like hook)
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Compare entered password with hashed password
userSchema.methods.matchPassword = async function (enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

### 3. `models/Client.js`

A client of the company who has debts/invoices.

```js
const mongoose = require('mongoose');

const clientSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Client name is required'],
      trim: true,
      minlength: 2,
      maxlength: 100,
    },
    email: {
      type: String,
      trim: true,
      lowercase: true,
    },
    phone: {
      type: String,
      trim: true,
    },
    address: {
      type: String,
      trim: true,
    },
    createdBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('Client', clientSchema);
```

### 4. `models/Invoice.js`

An unpaid invoice linked to a client.

```js
const mongoose = require('mongoose');

const invoiceSchema = new mongoose.Schema(
  {
    invoiceNumber: {
      type: String,
      required: [true, 'Invoice number is required'],
      unique: true,
      trim: true,
    },
    client: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Client',
      required: [true, 'Client is required'],
    },
    amount: {
      type: Number,
      required: [true, 'Amount is required'],
      min: [0, 'Amount cannot be negative'],
    },
    amountPaid: {
      type: Number,
      default: 0,
      min: 0,
    },
    dueDate: {
      type: Date,
      required: [true, 'Due date is required'],
    },
    status: {
      type: String,
      enum: ['unpaid', 'partially_paid', 'paid', 'overdue'],
      default: 'unpaid',
    },
    createdBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('Invoice', invoiceSchema);
```

### 5. `models/Payment.js`

A manual payment recorded against an invoice.

```js
const mongoose = require('mongoose');

const paymentSchema = new mongoose.Schema(
  {
    invoice: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Invoice',
      required: [true, 'Invoice is required'],
    },
    amount: {
      type: Number,
      required: [true, 'Payment amount is required'],
      min: [0.01, 'Payment must be greater than 0'],
    },
    paymentDate: {
      type: Date,
      default: Date.now,
    },
    paymentMethod: {
      type: String,
      enum: ['cash', 'check', 'transfer'],
      required: [true, 'Payment method is required'],
    },
    note: {
      type: String,
      trim: true,
      maxlength: 500,
    },
    recordedBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('Payment', paymentSchema);
```

### 6. `models/RecoveryAction.js`

A recovery/collection action taken on a client's debt.

```js
const mongoose = require('mongoose');

const recoveryActionSchema = new mongoose.Schema(
  {
    invoice: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Invoice',
      required: [true, 'Invoice is required'],
    },
    client: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Client',
      required: [true, 'Client is required'],
    },
    actionType: {
      type: String,
      enum: ['phone_call', 'email', 'letter', 'visit', 'legal'],
      required: [true, 'Action type is required'],
    },
    note: {
      type: String,
      trim: true,
      maxlength: 1000,
    },
    result: {
      type: String,
      trim: true,
      maxlength: 500,
    },
    actionDate: {
      type: Date,
      default: Date.now,
    },
    performedBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
  },
  {
    timestamps: true,
  }
);

module.exports = mongoose.model('RecoveryAction', recoveryActionSchema);
```

## Folder structure after this step

```
├── models/
│   ├── User.js
│   ├── Client.js
│   ├── Invoice.js
│   ├── Payment.js
│   └── RecoveryAction.js
├── utils/
│   └── HttpError.js
```

## Verify

```bash
npm run dev
# Server should still start without errors
# Models are just defined, not used yet — no runtime errors expected
```

## Commit

```bash
git add .
git commit -m "Adem: add all models"
```
