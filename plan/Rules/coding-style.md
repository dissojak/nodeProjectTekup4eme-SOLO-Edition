# Coding Style Rules

## Comments

**DO NOT over-comment.** The teacher will flag AI-generated code if every line has a comment.

### What to comment:
- `@desc`, `@route`, `@access` block above each controller function (3 lines only)
- Short one-liner before non-obvious logic (e.g., "// Hash password before saving")
- Nothing else

### What NOT to comment:
- Field definitions in models — the code is self-explanatory (type, required, enum speak for themselves)
- Obvious operations like `const { email, password } = req.body`
- Every single line of a function
- Inline comments on every property (no `// Remove whitespace` next to `trim: true`)

### Bad example (don't do this):
```js
// User email field
email: {
  type: String,       // Type is string
  required: true,     // Email is required
  unique: true,       // Email must be unique in the database
  trim: true,         // Remove whitespace
  lowercase: true,    // Convert to lowercase
},
```

### Good example (do this):
```js
email: {
  type: String,
  required: [true, 'Email is required'],
  unique: true,
  trim: true,
  lowercase: true,
},
```

## Controller documentation style

Every controller function gets exactly this block:
```js
// @desc    What it does
// @route   METHOD /api/resource/path
// @access  Public | Private | Private/Admin
const functionName = asyncHandler(async (req, res) => {
```

Only add inline comments when the logic is not obvious. One comment for a block, not per line.

## Error handling

- Use `HttpError` from `utils/HttpError.js` for throwing errors with status codes
- Use `express-async-handler` to wrap all async controller functions
- Use `express-validator` for input validation (not Joi)
- Return validation errors as: `return res.status(400).json({ errors: errors.array() });`

## Model style

- Keep models clean — no inline comments on every field
- Use Mongoose built-in validation messages: `required: [true, 'Field is required']`
- No extra blank lines between fields
- `timestamps: true` at the end

## Validators

- Clean chains, no comments between `.check()` calls
- Each validator is an exported array
- One file per resource (authValidator.js, clientValidator.js, etc.)

## Routes

- No comments in route files — the routes are self-documenting
- Import controller functions and middleware at the top
- Group public routes first, then protected routes

## General

- Use `const` by default, `let` only when reassignment is needed
- Use arrow functions for callbacks
- Use async/await (never .then() chains)
- Use destructuring for req.body
- No `console.log` in production code (remove debug logs before committing)
