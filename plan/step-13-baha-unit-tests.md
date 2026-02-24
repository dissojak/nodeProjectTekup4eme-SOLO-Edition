# Step 13 — Baha: Add Unit Tests

**Commit message:** `Baha: add unit tests`

---

## What to do

Add basic unit tests using **Jest** and **Supertest**. Tests cover authentication, client CRUD, and invoice CRUD. Uses an in-memory approach with test database.

## Setup

### 1. Update `package.json` — add Jest config

Add this to the root of `package.json`:

```json
{
  "jest": {
    "testEnvironment": "node",
    "testTimeout": 10000
  }
}
```

Make sure the test script is set:
```json
{
  "scripts": {
    "test": "jest --verbose --forceExit --detectOpenHandles"
  }
}
```

### 2. Create `.env.test`

A separate env file for testing (use a different database):

```env
PORT=5001
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/RECOUVRA_PLUS_TEST
JWT_SECRET=test_jwt_secret_key
JWT_EXPIRE=30d
NODE_ENV=test
```

> ⚠️ Use a DIFFERENT database name (e.g., `RECOUVRA_PLUS_TEST`) so tests don't affect production data.

## Files to create

### 1. `tests/setup.js`

Test setup — connects to test DB, provides helpers, and cleans up after.

```js
const mongoose = require('mongoose');
const dotenv = require('dotenv');

// Load test environment variables
dotenv.config({ path: '.env.test' });

const app = require('../app');

// Connect to test database before all tests
beforeAll(async () => {
  await mongoose.connect(process.env.MONGO_URI);
});

// Clean up database after each test suite
afterAll(async () => {
  // Drop test database collections
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
  await mongoose.connection.close();
});

module.exports = app;
```

### 2. `tests/auth.test.js`

```js
const request = require('supertest');
const app = require('./setup');

describe('Auth Endpoints', () => {
  let cookies;

  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'testuser@test.com',
          password: '123456',
          role: 'admin',
        });

      expect(res.statusCode).toBe(201);
      expect(res.body).toHaveProperty('_id');
      expect(res.body.name).toBe('Test User');
      expect(res.body.email).toBe('testuser@test.com');
      expect(res.body.role).toBe('admin');
      expect(res.body).not.toHaveProperty('password');
    });

    it('should not register with duplicate email', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User 2',
          email: 'testuser@test.com',
          password: '123456',
        });

      expect(res.statusCode).toBe(400);
    });

    it('should not register without required fields', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({});

      expect(res.statusCode).toBe(400);
    });

    it('should not register with short password', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Short Pass',
          email: 'short@test.com',
          password: '123',
        });

      expect(res.statusCode).toBe(400);
    });
  });

  describe('POST /api/auth/login', () => {
    it('should login with valid credentials', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'testuser@test.com',
          password: '123456',
        });

      expect(res.statusCode).toBe(200);
      expect(res.body).toHaveProperty('_id');
      expect(res.body.email).toBe('testuser@test.com');

      // Save cookies for authenticated requests
      cookies = res.headers['set-cookie'];
    });

    it('should not login with wrong password', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'testuser@test.com',
          password: 'wrongpassword',
        });

      expect(res.statusCode).toBe(401);
    });

    it('should not login with non-existent email', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'noone@test.com',
          password: '123456',
        });

      expect(res.statusCode).toBe(401);
    });
  });

  describe('GET /api/auth/me', () => {
    it('should get current user profile when authenticated', async () => {
      const res = await request(app)
        .get('/api/auth/me')
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body).toHaveProperty('_id');
      expect(res.body.email).toBe('testuser@test.com');
    });

    it('should return 401 when not authenticated', async () => {
      const res = await request(app).get('/api/auth/me');

      expect(res.statusCode).toBe(401);
    });
  });

  describe('POST /api/auth/logout', () => {
    it('should logout successfully', async () => {
      const res = await request(app)
        .post('/api/auth/logout')
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body.message).toBe('Logged out successfully');
    });
  });
});
```

### 3. `tests/client.test.js`

```js
const request = require('supertest');
const app = require('./setup');

describe('Client Endpoints', () => {
  let cookies;
  let clientId;

  // Login before running client tests
  beforeAll(async () => {
    // Register and login a user
    await request(app)
      .post('/api/auth/register')
      .send({
        name: 'Client Tester',
        email: 'clienttester@test.com',
        password: '123456',
        role: 'manager',
      });

    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'clienttester@test.com',
        password: '123456',
      });

    cookies = loginRes.headers['set-cookie'];
  });

  describe('POST /api/clients', () => {
    it('should create a new client', async () => {
      const res = await request(app)
        .post('/api/clients')
        .set('Cookie', cookies)
        .send({
          name: 'Test Company',
          email: 'company@test.com',
          phone: '12345678',
          address: '123 Test Street',
        });

      expect(res.statusCode).toBe(201);
      expect(res.body).toHaveProperty('_id');
      expect(res.body.name).toBe('Test Company');
      clientId = res.body._id;
    });

    it('should not create client without name', async () => {
      const res = await request(app)
        .post('/api/clients')
        .set('Cookie', cookies)
        .send({
          email: 'noname@test.com',
        });

      expect(res.statusCode).toBe(400);
    });

    it('should not create client when not authenticated', async () => {
      const res = await request(app)
        .post('/api/clients')
        .send({ name: 'No Auth Client' });

      expect(res.statusCode).toBe(401);
    });
  });

  describe('GET /api/clients', () => {
    it('should get all clients', async () => {
      const res = await request(app)
        .get('/api/clients')
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(Array.isArray(res.body)).toBe(true);
      expect(res.body.length).toBeGreaterThan(0);
    });
  });

  describe('GET /api/clients/:id', () => {
    it('should get a client by ID', async () => {
      const res = await request(app)
        .get(`/api/clients/${clientId}`)
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body._id).toBe(clientId);
      expect(res.body.name).toBe('Test Company');
    });

    it('should return 404 for non-existent client', async () => {
      const res = await request(app)
        .get('/api/clients/507f1f77bcf86cd799439011')
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(404);
    });
  });

  describe('PUT /api/clients/:id', () => {
    it('should update a client', async () => {
      const res = await request(app)
        .put(`/api/clients/${clientId}`)
        .set('Cookie', cookies)
        .send({ name: 'Updated Company' });

      expect(res.statusCode).toBe(200);
      expect(res.body.name).toBe('Updated Company');
    });
  });

  describe('DELETE /api/clients/:id', () => {
    it('should delete a client', async () => {
      const res = await request(app)
        .delete(`/api/clients/${clientId}`)
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body.message).toBe('Client deleted successfully');
    });
  });
});
```

### 4. `tests/invoice.test.js`

```js
const request = require('supertest');
const app = require('./setup');

describe('Invoice Endpoints', () => {
  let cookies;
  let clientId;
  let invoiceId;

  // Setup: create user and client
  beforeAll(async () => {
    await request(app)
      .post('/api/auth/register')
      .send({
        name: 'Invoice Tester',
        email: 'invoicetester@test.com',
        password: '123456',
        role: 'manager',
      });

    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'invoicetester@test.com',
        password: '123456',
      });

    cookies = loginRes.headers['set-cookie'];

    // Create a client for invoices
    const clientRes = await request(app)
      .post('/api/clients')
      .set('Cookie', cookies)
      .send({ name: 'Invoice Test Client' });

    clientId = clientRes.body._id;
  });

  describe('POST /api/invoices', () => {
    it('should create a new invoice', async () => {
      const res = await request(app)
        .post('/api/invoices')
        .set('Cookie', cookies)
        .send({
          invoiceNumber: 'INV-TEST-001',
          client: clientId,
          amount: 1500,
          dueDate: '2026-06-01',
        });

      expect(res.statusCode).toBe(201);
      expect(res.body).toHaveProperty('_id');
      expect(res.body.invoiceNumber).toBe('INV-TEST-001');
      expect(res.body.amount).toBe(1500);
      expect(res.body.status).toBe('unpaid');
      invoiceId = res.body._id;
    });

    it('should not create invoice with duplicate number', async () => {
      const res = await request(app)
        .post('/api/invoices')
        .set('Cookie', cookies)
        .send({
          invoiceNumber: 'INV-TEST-001',
          client: clientId,
          amount: 500,
          dueDate: '2026-06-01',
        });

      expect(res.statusCode).toBe(400);
    });

    it('should not create invoice without required fields', async () => {
      const res = await request(app)
        .post('/api/invoices')
        .set('Cookie', cookies)
        .send({});

      expect(res.statusCode).toBe(400);
    });
  });

  describe('GET /api/invoices', () => {
    it('should get all invoices', async () => {
      const res = await request(app)
        .get('/api/invoices')
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(Array.isArray(res.body)).toBe(true);
    });
  });

  describe('GET /api/invoices/:id', () => {
    it('should get invoice by ID', async () => {
      const res = await request(app)
        .get(`/api/invoices/${invoiceId}`)
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body.invoiceNumber).toBe('INV-TEST-001');
    });
  });

  describe('GET /api/invoices/client/:clientId', () => {
    it('should get invoices by client', async () => {
      const res = await request(app)
        .get(`/api/invoices/client/${clientId}`)
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(Array.isArray(res.body)).toBe(true);
      expect(res.body.length).toBeGreaterThan(0);
    });
  });

  describe('PUT /api/invoices/:id', () => {
    it('should update an invoice', async () => {
      const res = await request(app)
        .put(`/api/invoices/${invoiceId}`)
        .set('Cookie', cookies)
        .send({ amount: 2000 });

      expect(res.statusCode).toBe(200);
      expect(res.body.amount).toBe(2000);
    });
  });

  describe('DELETE /api/invoices/:id', () => {
    it('should delete an invoice', async () => {
      const res = await request(app)
        .delete(`/api/invoices/${invoiceId}`)
        .set('Cookie', cookies);

      expect(res.statusCode).toBe(200);
      expect(res.body.message).toBe('Invoice deleted successfully');
    });
  });
});
```

## Folder structure after this step

```
├── tests/
│   ├── setup.js         ← NEW (shared test config)
│   ├── auth.test.js     ← NEW
│   ├── client.test.js   ← NEW
│   └── invoice.test.js  ← NEW
├── .env.test            ← NEW
```

## Verify

```bash
npm test

# Expected output:
# PASS  tests/auth.test.js
#   Auth Endpoints
#     POST /api/auth/register
#       ✓ should register a new user
#       ✓ should not register with duplicate email
#       ✓ should not register without required fields
#       ✓ should not register with short password
#     POST /api/auth/login
#       ✓ should login with valid credentials
#       ✓ should not login with wrong password
#       ✓ should not login with non-existent email
#     GET /api/auth/me
#       ✓ should get current user profile when authenticated
#       ✓ should return 401 when not authenticated
#     POST /api/auth/logout
#       ✓ should logout successfully
#
# PASS  tests/client.test.js
#   ... (similar)
#
# PASS  tests/invoice.test.js
#   ... (similar)
#
# Test Suites: 3 passed, 3 total
# Tests:       ~20 passed, ~20 total
```

## Commit

```bash
git add .
git commit -m "Baha: add unit tests"
```
