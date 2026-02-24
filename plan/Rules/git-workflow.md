# Git Workflow Rules

## Commit messages

Every commit MUST be prefixed with the developer name:

- **Adem's commits:** `Adem: description here`
- **Baha's commits:** `Baha: description here`

Use the exact commit message from the corresponding `plan/step-XX-*.md` file.

### Commit message mapping (remaining steps):

| Step | Prefix | Commit message |
|------|--------|----------------|
| 5    | Baha   | `Baha: add error handling and validation middleware` |
| 6    | Adem   | `Adem: implement client controller and routes` |
| 7    | Baha   | `Baha: implement invoice controller and routes` |
| 8    | Adem   | `Adem: implement payment controller and routes` |
| 9    | Baha   | `Baha: implement recovery action controller and routes` |
| 10   | Baha   | `Baha: implement user management controller and routes` |
| 11   | Baha   | `Baha: implement stats controller and routes` |
| 12   | Adem   | `Adem: add Swagger API documentation` |
| 13   | Baha   | `Baha: add unit tests` |
| 14   | Adem   | `Adem: add README documentation` |

## Process per step

1. Read the plan file for the step (`plan/step-XX-*.md`)
2. Create/edit files exactly as described
3. Verify the server starts with `npm start`
4. Stage all changes: `git add .`
5. Commit with the exact message from the table above
6. Push: `git push origin main`

## Git config

The git user is already configured. Do not change it.

## Branch

Everything goes on `main`. No feature branches needed.

## What NOT to commit

- `node_modules/` (in .gitignore)
- `.env` (in .gitignore)
- No plan files should be modified during implementation
