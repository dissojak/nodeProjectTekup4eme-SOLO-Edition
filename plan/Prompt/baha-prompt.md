# Baha's Prompt — Continue Recouvra+ API

Copy this entire prompt and give it to your AI agent. Attach/expand the files and folders mentioned below.

---

## Prompt to paste:

```
I'm working on a Node.js REST API project called "Recouvra+" (debt collection management // u can read this file Recouvra__API_de_gestion_du_recouvremen.pdf ). My teammate Adem already completed steps 1-4. I need to continue from step 5.

## What's already done (DO NOT redo these):
- Step 1: Project setup (Express, Mongoose, dependencies, folder structure)
- Step 2: All 5 models (User, Client, Invoice, Payment, RecoveryAction) + HttpError utility
- Step 3: JWT auth middleware (protect, authorize) + generateToken utility
- Step 4: Auth controller & routes (register, login, logout, getMe)

## My steps to implement (in order):
1. Step 5: Error handling and validation middleware → `Baha: add error handling and validation middleware`
2. Step 7: Invoice controller and routes → `Baha: implement invoice controller and routes`
3. Step 9: Recovery action controller and routes → `Baha: implement recovery action controller and routes`
4. Step 10: User management controller and routes → `Baha: implement user management controller and routes`
5. Step 11: Stats controller and routes → `Baha: implement stats controller and routes`
6. Step 13: Unit tests → `Baha: add unit tests`

## IMPORTANT — Between my steps, Adem will do his own commits:
- After my Step 5, Adem does Step 6 (client controller)
- After my Step 7, Adem does Step 8 (payment controller)
- Steps 9, 10, 11 are mine in a row
- After my Step 11, Adem does Step 12 (swagger)
- Then I do Step 13 (tests), then Adem does Step 14 (readme)

So I need to implement ONLY my steps. But some of my steps depend on Adem's work being done first. Do one step at a time, in order.

## Rules — READ THESE BEFORE WRITING ANY CODE:

Read all 3 files in the `plan/Rules/` folder carefully:
- `plan/Rules/coding-style.md` — How to write code (minimal comments, no over-commenting)
- `plan/Rules/git-workflow.md` — Commit messages must start with "Baha:" 
- `plan/Rules/architecture.md` — Project structure and conventions

Key rules summary:
- NO over-commenting. Don't comment every line. Teacher will flag it as AI-generated.
- Controllers get ONLY `@desc`, `@route`, `@access` header (3 lines). Maybe 1-2 inline comments for non-obvious logic.
- Models, validators, routes = NO comments (the code speaks for itself)
- Use `HttpError` from `utils/HttpError.js` for errors
- Use `express-async-handler` to wrap async controllers
- Use `express-validator` for validation (NOT Joi)
- Return validation errors as: `res.status(400).json({ errors: errors.array() })`
- All commit messages start with `Baha:` — use the exact message from the plan file

## Process for each step:
1. Read the plan file for that step (e.g., `plan/step-05-baha-error-validation-middleware.md`)
2. Look at existing code to understand patterns (check `controllers/authController.js`, `routes/auth.js` as reference)
3. Create the files described in the plan
4. Register new routes in `app.js` under the routes section
5. Verify server starts: `npm start`
6. Commit: `git add . && git commit -m "Baha: exact message from plan"`
7. Push: `git push origin main`

## Start with Step 5 now.

Read `plan/step-05-baha-error-validation-middleware.md` and implement it.
```

---

## What to attach/expand when giving this prompt:

### Files to expand (click to show content):
- `plan/Rules/coding-style.md`
- `plan/Rules/git-workflow.md`  
- `plan/Rules/architecture.md`
- `plan/step-05-baha-error-validation-middleware.md` (the step you're starting with)
- `plan/Ticket/Recouvra__API_de_gestion_du_recouvremen.pdf` (the project description)

### Files to expand as reference (so AI sees existing code style):
- `controllers/authController.js`
- `middleware/authMiddleware.js`
- `routes/auth.js`
- `validators/authValidator.js`
- `models/User.js` (as model reference)
- `app.js`
- `utils/HttpError.js`

### Folder to attach:
- The repo itself (GitHub repo attachment if available)

---

## After each step:

When a step is done and committed, for the NEXT step just say:

```
Now do Step X. Read `plan/step-XX-*.md` and implement it. Follow the same rules.
```

And expand/attach the plan file for that step.

---

## If Adem needs to do a step between yours:

Tell your AI to stop. Let Adem push his commit. Pull with `git pull origin main`, then continue with your next step.
