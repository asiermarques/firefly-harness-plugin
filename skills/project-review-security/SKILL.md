---
name: project-review-security
description: Security review of the project or of recent changes — input validation, per-user/tenant data isolation, authorization gaps, API abuse, XSS, injection, unsafe file/URL handling, and sensitive data exposure. Use when reviewing code for security, or as the security dimension of project-review-methodology.
user-invocable: false
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git status *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git show *)
---

# Project security review

Review security risks in the code in scope (the caller's scope, or the current changes from `git diff` if none is given). This skill **reviews only** — don't fix code unless explicitly asked.

## Required context

Read first (skip silently any that don't exist): `CLAUDE.md`, `docs/PRODUCT.md`, `docs/ARCHITECTURE.md`, `docs/adr/*.md`.

### Establish the threat model

Before reviewing, determine from the docs — and confirm in the code — and state at the top of your output:

- **Auth model**: authenticated, unauthenticated multi-user (e.g. user picked on the client), single-user, or multi-tenant.
- **Data scoping**: how data is tied to a user/tenant and where that scope is enforced.
- **Persistence**: database and data-access style (raw SQL, query builder, ORM).
- **Trust boundaries**: where untrusted input enters (HTTP, files, webhooks, queues) and where user-controlled data is rendered.

If the docs are silent, infer from the code and mark it as an assumption. Don't assume a model the project doesn't have.

## Security checks

Review for:

- Missing input validation, or validation only on the client.
- One user (or tenant) reading or modifying another's data.
- Missing authorization checks — and, in unauthenticated multi-user apps, missing scope checks that play the role of authorization (e.g. trusting a client-supplied user ID without verifying the resource belongs to it).
- API endpoints without abuse controls (rate limits, size limits, pagination) where relevant.
- Sensitive data exposed in API responses, logs, errors, or client state.
- XSS, including unsafe HTML rendering.
- Injection (database, command, template, path), especially raw queries or string interpolation.
- Unsafe file, URL, redirect, date, or ID handling.
- Overly detailed error messages.
- Secrets in code or insecure environment variable usage.
- Dependency or package risks, if visible from project files.

## Database / data-access checks

- Parameterized queries over string-built ones; flag raw queries unless clearly safe.
- User/tenant scoping present in query filters.
- Joins that could leak cross-user data.
- Mutations constrained by ownership or scope, not just by ID.

## Frontend rendering checks

- Raw/unescaped HTML injected into the DOM.
- Untrusted data rendered without escaping.
- Unsafe URL generation (e.g. `javascript:` links from user data).
- Client-only checks that must also be enforced server-side.

## Output

Return security findings only, most-severe first, using the severity scale of `firefly:project-review-methodology` (Critical / High / Medium / Low). Verify each finding against the code before reporting it.

For each finding:

- **Severity:**
- **File/location:** `path:line`
- **Issue:**
- **Why it matters:** the concrete exploit or failure scenario
- **Recommended fix:**

If nothing in scope has a security issue, say so explicitly along with the threat model you assumed.
