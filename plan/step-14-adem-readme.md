# Step 14 — Adem: Add README

**Commit message:** `Adem: add README`

---

## What to do

Create a comprehensive README.md documenting the project, setup instructions, API endpoints, design patterns used, and team contributions.

## Files to create

### `README.md`

```markdown
# Recouvra+ — API de Gestion du Recouvrement

API REST développée avec Express.js pour gérer les clients, les factures impayées et les actions de recouvrement d'une entreprise.

## Table des matières

- [Technologies](#technologies)
- [Installation](#installation)
- [Configuration](#configuration)
- [Lancement](#lancement)
- [API Endpoints](#api-endpoints)
- [Design Patterns](#design-patterns)
- [Tests](#tests)
- [Documentation Swagger](#documentation-swagger)
- [Structure du projet](#structure-du-projet)
- [Équipe](#équipe)

## Technologies

- **Node.js** 22
- **Express.js** — Framework web
- **MongoDB** + **Mongoose** — Base de données NoSQL
- **JWT** (jsonwebtoken) — Authentification
- **express-validator** — Validation des données
- **Swagger** (swagger-jsdoc + swagger-ui-express) — Documentation API
- **Jest** + **Supertest** — Tests unitaires
- **bcryptjs** — Hashage des mots de passe
- **Helmet** — Sécurité HTTP headers
- **Morgan** — Logging HTTP

## Installation

```bash
# Cloner le projet
git clone <repository-url>
cd recouvra-plus

# Installer les dépendances
npm install
```

## Configuration

Créer un fichier `.env` à la racine du projet :

```env
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/RECOUVRA_PLUS
JWT_SECRET=votre_clé_secrète
JWT_EXPIRE=30d
NODE_ENV=development
```

## Lancement

```bash
# Mode développement (avec nodemon)
npm run dev

# Mode production
npm start
```

Le serveur démarre sur `http://localhost:5000`.

## API Endpoints

### Authentification (`/api/auth`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| POST | `/api/auth/register` | Inscription | Public |
| POST | `/api/auth/login` | Connexion | Public |
| POST | `/api/auth/logout` | Déconnexion | Authentifié |
| GET | `/api/auth/me` | Profil courant | Authentifié |

### Utilisateurs (`/api/users`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/users` | Liste des utilisateurs | Admin |
| GET | `/api/users/:id` | Détail utilisateur | Admin, Manager |
| PUT | `/api/users/:id` | Modifier utilisateur | Admin |
| DELETE | `/api/users/:id` | Supprimer utilisateur | Admin |

### Clients (`/api/clients`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/clients` | Liste des clients | Tous |
| GET | `/api/clients/:id` | Détail client | Tous |
| POST | `/api/clients` | Créer un client | Agent, Manager |
| PUT | `/api/clients/:id` | Modifier client | Agent, Manager |
| DELETE | `/api/clients/:id` | Supprimer client | Manager, Admin |

### Factures (`/api/invoices`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/invoices` | Liste des factures | Tous |
| GET | `/api/invoices/:id` | Détail facture | Tous |
| GET | `/api/invoices/client/:clientId` | Factures par client | Tous |
| POST | `/api/invoices` | Créer une facture | Agent, Manager |
| PUT | `/api/invoices/:id` | Modifier facture | Agent, Manager |
| DELETE | `/api/invoices/:id` | Supprimer facture | Manager, Admin |

### Paiements (`/api/payments`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/payments` | Liste des paiements | Tous |
| GET | `/api/payments/invoice/:invoiceId` | Paiements par facture | Tous |
| POST | `/api/payments` | Enregistrer un paiement | Agent, Manager |

### Actions de Recouvrement (`/api/recovery-actions`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/recovery-actions` | Liste des actions | Tous |
| GET | `/api/recovery-actions/client/:clientId` | Actions par client | Tous |
| GET | `/api/recovery-actions/invoice/:invoiceId` | Actions par facture | Tous |
| POST | `/api/recovery-actions` | Créer une action | Agent, Manager |
| PUT | `/api/recovery-actions/:id` | Modifier action | Agent, Manager |
| DELETE | `/api/recovery-actions/:id` | Supprimer action | Manager, Admin |

### Statistiques (`/api/stats`)

| Méthode | Endpoint | Description | Accès |
|---------|----------|-------------|-------|
| GET | `/api/stats/overview` | Vue d'ensemble | Manager, Admin |
| GET | `/api/stats/invoices` | Statistiques factures | Manager, Admin |
| GET | `/api/stats/agents` | Performance agents | Manager, Admin |

## Design Patterns

| Pattern | Utilisation | Fichiers |
|---------|-------------|----------|
| **MVC** | Séparation models/controllers/routes | Toute l'architecture |
| **Singleton** | Connexion unique à MongoDB | `config/db.js` |
| **Chain of Responsibility** | Chaîne de middlewares (auth → validate → controller → error) | `middleware/` |
| **Decorator** | `asyncHandler` pour gestion d'erreurs async | Tous les controllers |
| **Factory** | Création d'erreurs HTTP personnalisées | `utils/HttpError.js` |
| **Strategy** | Traitement différent par méthode de paiement | `strategies/paymentStrategies.js` |

## Tests

```bash
# Lancer les tests
npm test

# Les tests couvrent :
# - Authentification (register, login, logout, profil)
# - Gestion des clients (CRUD)
# - Gestion des factures (CRUD)
```

> **Note :** Créer un fichier `.env.test` avec une base de données de test séparée.

## Documentation Swagger

La documentation interactive est disponible à : `http://localhost:5000/api-docs`

## Structure du projet

```
├── config/
│   ├── db.js                  # Connexion MongoDB (Singleton)
│   └── swagger.js             # Configuration Swagger
├── controllers/
│   ├── authController.js      # Authentification
│   ├── userController.js      # Gestion utilisateurs
│   ├── clientController.js    # Gestion clients
│   ├── invoiceController.js   # Gestion factures
│   ├── paymentController.js   # Gestion paiements
│   ├── recoveryActionController.js  # Actions de recouvrement
│   └── statsController.js     # Statistiques
├── middleware/
│   ├── authMiddleware.js      # Protection JWT + autorisation par rôle
│   ├── errorMiddleware.js     # Gestion globale des erreurs
│   └── validateMiddleware.js  # Validation express-validator
├── models/
│   ├── User.js
│   ├── Client.js
│   ├── Invoice.js
│   ├── Payment.js
│   └── RecoveryAction.js
├── routes/
│   ├── auth.js
│   ├── users.js
│   ├── clients.js
│   ├── invoices.js
│   ├── payments.js
│   ├── recoveryActions.js
│   └── stats.js
├── strategies/
│   └── paymentStrategies.js   # Strategy pattern pour paiements
├── validators/
│   ├── authValidator.js
│   ├── clientValidator.js
│   ├── invoiceValidator.js
│   ├── paymentValidator.js
│   └── recoveryActionValidator.js
├── utils/
│   ├── generateToken.js       # Génération JWT + cookie
│   └── HttpError.js           # Classe d'erreur personnalisée
├── tests/
│   ├── setup.js
│   ├── auth.test.js
│   ├── client.test.js
│   └── invoice.test.js
├── app.js                     # Configuration Express
├── server.js                  # Point d'entrée
├── .env                       # Variables d'environnement
├── .env.test                  # Variables pour tests
├── .gitignore
├── package.json
└── README.md
```

## Rôles et Permissions

| Action | Agent | Manager | Admin |
|--------|:-----:|:-------:|:-----:|
| Voir clients/factures/paiements | ✅ | ✅ | ✅ |
| Créer/modifier clients | ✅ | ✅ | ❌ |
| Créer/modifier factures | ✅ | ✅ | ❌ |
| Enregistrer paiements | ✅ | ✅ | ❌ |
| Supprimer clients/factures | ❌ | ✅ | ✅ |
| Voir statistiques | ❌ | ✅ | ✅ |
| Gérer utilisateurs | ❌ | ❌ | ✅ |

## Équipe

- **Adem** — Setup, Models, Auth, Client, Payment, Swagger, README
- **Baha** — Error Middleware, Invoice, Recovery Actions, User Management, Stats, Tests
```

## Verify

```bash
# The README should render properly on GitHub
# Check that all links and tables display correctly

# Final verification — everything works:
npm run dev     # Server starts
npm test        # Tests pass
# Visit http://localhost:5000/api-docs — Swagger loads
```

## Commit

```bash
git add .
git commit -m "Adem: add README"
```

---

## 🎉 PROJECT COMPLETE

After this commit, your git log should look like:

```
Adem: add README
Baha: add unit tests
Adem: add swagger documentation
Baha: implement stats controller and routes
Baha: implement user management controller and routes
Baha: implement recovery action controller and routes
Adem: implement payment controller and routes
Baha: implement invoice controller and routes
Adem: implement client controller and routes
Baha: add error and validation middleware
Adem: implement auth controller and routes
Adem: add JWT auth and middleware
Adem: add all models
Adem: project structure and setup
```

**14 commits total: 8 Adem / 6 Baha**
