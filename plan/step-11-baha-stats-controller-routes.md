# Step 11 — Baha: Implement Stats Controller and Routes

**Commit message:** `Baha: implement stats controller and routes`

---

## What to do

Create statistics/analytics endpoints. Provides overview dashboard data, invoice statistics, and agent performance stats. Only accessible by managers and admins.

## Files to create

### 1. `controllers/statsController.js`

Uses MongoDB aggregation pipeline for computing statistics.

```js
const asyncHandler = require('express-async-handler');
const Invoice = require('../models/Invoice');
const Payment = require('../models/Payment');
const Client = require('../models/Client');
const RecoveryAction = require('../models/RecoveryAction');
const User = require('../models/User');

// @desc    Get overview statistics
// @route   GET /api/stats/overview
// @access  Private (manager, admin)
const getOverview = asyncHandler(async (req, res) => {
  const totalClients = await Client.countDocuments();
  const totalInvoices = await Invoice.countDocuments();
  const totalPayments = await Payment.countDocuments();
  const totalRecoveryActions = await RecoveryAction.countDocuments();

  // Invoice status breakdown
  const invoicesByStatus = await Invoice.aggregate([
    {
      $group: {
        _id: '$status',
        count: { $sum: 1 },
        totalAmount: { $sum: '$amount' },
      },
    },
  ]);

  // Total amounts
  const totalAmountDue = await Invoice.aggregate([
    { $group: { _id: null, total: { $sum: '$amount' } } },
  ]);

  const totalAmountPaid = await Invoice.aggregate([
    { $group: { _id: null, total: { $sum: '$amountPaid' } } },
  ]);

  res.json({
    totalClients,
    totalInvoices,
    totalPayments,
    totalRecoveryActions,
    invoicesByStatus,
    totalAmountDue: totalAmountDue[0]?.total || 0,
    totalAmountPaid: totalAmountPaid[0]?.total || 0,
    totalAmountRemaining:
      (totalAmountDue[0]?.total || 0) - (totalAmountPaid[0]?.total || 0),
  });
});

// @desc    Get invoice statistics
// @route   GET /api/stats/invoices
// @access  Private (manager, admin)
const getInvoiceStats = asyncHandler(async (req, res) => {
  // Invoices by status
  const byStatus = await Invoice.aggregate([
    {
      $group: {
        _id: '$status',
        count: { $sum: 1 },
        totalAmount: { $sum: '$amount' },
        totalPaid: { $sum: '$amountPaid' },
      },
    },
  ]);

  // Overdue invoices (unpaid or partially_paid + past due date)
  const overdueInvoices = await Invoice.find({
    status: { $in: ['unpaid', 'partially_paid'] },
    dueDate: { $lt: new Date() },
  })
    .populate('client', 'name email')
    .sort({ dueDate: 1 });

  // Invoices created per month (last 12 months)
  const twelveMonthsAgo = new Date();
  twelveMonthsAgo.setMonth(twelveMonthsAgo.getMonth() - 12);

  const invoicesPerMonth = await Invoice.aggregate([
    { $match: { createdAt: { $gte: twelveMonthsAgo } } },
    {
      $group: {
        _id: {
          year: { $year: '$createdAt' },
          month: { $month: '$createdAt' },
        },
        count: { $sum: 1 },
        totalAmount: { $sum: '$amount' },
      },
    },
    { $sort: { '_id.year': 1, '_id.month': 1 } },
  ]);

  // Payment methods distribution
  const paymentMethodStats = await Payment.aggregate([
    {
      $group: {
        _id: '$paymentMethod',
        count: { $sum: 1 },
        totalAmount: { $sum: '$amount' },
      },
    },
  ]);

  res.json({
    byStatus,
    overdueInvoices,
    overdueCount: overdueInvoices.length,
    invoicesPerMonth,
    paymentMethodStats,
  });
});

// @desc    Get agent performance statistics
// @route   GET /api/stats/agents
// @access  Private (manager, admin)
const getAgentStats = asyncHandler(async (req, res) => {
  // Recovery actions per agent
  const actionsByAgent = await RecoveryAction.aggregate([
    {
      $group: {
        _id: '$performedBy',
        totalActions: { $sum: 1 },
        actionTypes: { $push: '$actionType' },
      },
    },
    {
      $lookup: {
        from: 'users',
        localField: '_id',
        foreignField: '_id',
        as: 'agent',
      },
    },
    { $unwind: '$agent' },
    {
      $project: {
        agentName: '$agent.name',
        agentEmail: '$agent.email',
        totalActions: 1,
        actionTypes: 1,
      },
    },
    { $sort: { totalActions: -1 } },
  ]);

  // Payments recorded per agent
  const paymentsByAgent = await Payment.aggregate([
    {
      $group: {
        _id: '$recordedBy',
        totalPaymentsRecorded: { $sum: 1 },
        totalAmountCollected: { $sum: '$amount' },
      },
    },
    {
      $lookup: {
        from: 'users',
        localField: '_id',
        foreignField: '_id',
        as: 'agent',
      },
    },
    { $unwind: '$agent' },
    {
      $project: {
        agentName: '$agent.name',
        agentEmail: '$agent.email',
        totalPaymentsRecorded: 1,
        totalAmountCollected: 1,
      },
    },
    { $sort: { totalAmountCollected: -1 } },
  ]);

  res.json({
    actionsByAgent,
    paymentsByAgent,
  });
});

module.exports = { getOverview, getInvoiceStats, getAgentStats };
```

### 2. `routes/stats.js`

```js
const express = require('express');
const router = express.Router();
const {
  getOverview,
  getInvoiceStats,
  getAgentStats,
} = require('../controllers/statsController');
const { protect, authorize } = require('../middleware/authMiddleware');

router.use(protect);
router.use(authorize('manager', 'admin'));

router.get('/overview', getOverview);
router.get('/invoices', getInvoiceStats);
router.get('/agents', getAgentStats);

module.exports = router;
```

### 3. Update `app.js` — add stats routes

Add this line in the routes section of `app.js`:

```js
const statsRoutes = require('./routes/stats');
app.use('/api/stats', statsRoutes);
```

## Folder structure after this step

```
├── controllers/
│   ├── ...all previous controllers...
│   └── statsController.js    ← NEW
├── routes/
│   ├── ...all previous routes...
│   └── stats.js              ← NEW
```

## Verify

```bash
npm run dev

# Must be logged in as manager or admin:

# Overview stats:
curl http://localhost:5000/api/stats/overview -b cookies.txt

# Invoice stats:
curl http://localhost:5000/api/stats/invoices -b cookies.txt

# Agent performance:
curl http://localhost:5000/api/stats/agents -b cookies.txt
```

## Commit

```bash
git add .
git commit -m "Baha: implement stats controller and routes"
```
