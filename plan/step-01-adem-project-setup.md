# Step 1 — Adem: Project Structure and Setup

**Commit message:** `Adem: project structure and setup`

---

## What to do

Initialize the project, install all dependencies, and create the base configuration files.

## Files to create

### 1. `package.json` (via npm init)

```bash
cd /Users/stoon/Desktop/Projects/nodeProjectTekup4eme
npm init -y
```

Then install dependencies:

```bash
# Production dependencies
npm install express mongoose dotenv cors cookie-parser helmet morgan jsonwebtoken bcryptjs express-validator express-async-handler

# Dev dependencies
npm install --save-dev nodemon jest supertest
```

Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest --verbose --forceExit --detectOpenHandles"
  }
}
```

### 2. `.gitignore`

```
node_modules/
.env
```

### 3. `.env`

```env
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/RECOUVRA_PLUS
JWT_SECRET=your_jwt_secret_key_here
JWT_EXPIRE=30d
NODE_ENV=development
```

> ⚠️ Replace `MONGO_URI` with your actual MongoDB Atlas connection string.

### 4. `config/db.js`

**Design pattern:** Singleton — one shared connection reused across the app.

```js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
    console.log(`MongoDB connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 5. `app.js`

**Design pattern:** Chain of Responsibility — request flows through middleware chain.

```js
const express = require('express');
const cors = require('cors');
const cookieParser = require('cookie-parser');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();

// Security middleware
app.use(helmet());

// CORS
app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true,
}));

// Logging (dev only)
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Cookie parsing
app.use(cookieParser());

// ====== ROUTES WILL BE ADDED HERE IN LATER STEPS ======

// Root route
app.get('/', (req, res) => {
  res.json({ message: 'Welcome to Recouvra+ API' });
});

module.exports = app;
```

### 6. `server.js`

**Why separate from app.js?** So we can import `app` in tests (supertest) without starting the server.

```js
const dotenv = require('dotenv');
dotenv.config();

const app = require('./app');
const connectDB = require('./config/db');

const PORT = process.env.PORT || 5000;

connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
  });
});
```

### 7. Create empty folders

```bash
mkdir -p controllers models routes middleware validators utils strategies tests
```

## Folder structure after this step

```
├── config/
│   └── db.js
├── controllers/       (empty)
├── middleware/         (empty)
├── models/            (empty)
├── routes/            (empty)
├── strategies/        (empty)
├── tests/             (empty)
├── utils/             (empty)
├── validators/        (empty)
├── app.js
├── server.js
├── .env
├── .gitignore
├── package.json
```

## Verify

```bash
npm run dev
# Should output: "Server running in development mode on port 5500"
# And: "MongoDB connected: <host>" (if MONGO_URI is set correctly)
```

## Commit

```bash
git init
git add .
git commit -m "Adem: project structure and setup"
```
