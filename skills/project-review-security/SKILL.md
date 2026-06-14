---
name: project-review-security
description: Review security risks in the project, especially input validation, unauthenticated multi-user data isolation, API abuse, XSS, injection safety, and sensitive data exposure.
user-invocable: false
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Project security review

Use this skill to review security risks in the current project.

## Required context

Read first:

- `docs/PRODUCT.md`
- `docs/ARCHITECTURE.md`

Remember:

- The app is unauthenticated.
- Multiple users can exist.
- User-scoped data isolation is critical.
- The backend persists data through a database (directly or via an ORM/query builder).
- The frontend renders user-controlled data in a UI.

## Security checks

Review for:

- Missing input validation.
- Missing backend validation when frontend validation exists.
- User A accessing or modifying user B data.
- Unsafe assumptions in the unauthenticated multi-user model.
- API endpoints without abuse controls where relevant.
- Sensitive data exposed in API responses, logs, errors, or client state.
- XSS risks, including unsafe HTML rendering.
- Injection risks (database, command, etc.), especially raw queries or unsafe interpolation.
- Unsafe file, URL, date, or ID handling.
- Overly detailed error messages.
- Insecure environment variable usage.
- Dependency or package risks if visible from project files.
- Missing authorization-like checks even without formal auth.

## Database / data-access checks

- Prefer parameterized queries over string-built ones.
- Flag raw queries unless clearly safe.
- Check user/resource scoping in query filters.
- Check joins do not leak cross-user data.
- Check mutations include ownership or scope constraints.

## Frontend rendering checks

- Flag raw/unescaped HTML injection into the DOM.
- Check untrusted data rendering.
- Check unsafe URL generation.
- Check client-only assumptions that should be enforced server-side.

## Output

Return security findings only.

For each finding, include:

- Severity:
- File/location:
- Issue:
- Why it matters:
- Recommended fix:

Do not fix code unless explicitly asked.
