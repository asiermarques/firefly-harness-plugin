---
name: project-review-testing
description: Review test coverage and test quality for the project or recent changes — missing edge cases and error paths, weak or over-mocked tests, integration gaps, and vertical-slice completeness. Use when reviewing whether code is sufficiently tested, or as the testing dimension of project-review-methodology.
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

# Project test review

Review whether the code in scope (the caller's scope, or the current changes from `git diff` if none is given) is sufficiently and meaningfully tested. This skill **reviews only** — don't write tests unless explicitly asked.

## Required context

Read first (skip silently any that don't exist): `CLAUDE.md`, `docs/PRODUCT.md`, `docs/ARCHITECTURE.md`. If the code came from a `.workflow/` user story, read its acceptance criteria too — they are the minimum the tests must prove.

From them and from the existing tests, establish before reviewing:

- **Where tests live and how they're named** (co-located, `__tests__/`, `tests/`, `*.spec.*`…). Don't assume a layout — follow the project's.
- **Which test levels exist** (unit, integration, component, e2e) and what each is used for.
- **The product rules** the tests must enforce (business rules, deliberate constraints).

Good tests cover behavior, not implementation details.

## Review checks

Look for missing or weak tests around:

- Empty states, null/undefined values.
- Invalid inputs and boundary values.
- Duplicate entities.
- Invalid dates and time-zone edges, where dates are involved.
- Network or API failures.
- Backend validation failures.
- Per-user/tenant data isolation and multi-user interactions, if the app has more than one user.
- Concurrent or repeated operations (double submit, retries, idempotency).
- UI error and loading states, where relevant.
- Each acceptance criterion of the related user story, if any.
- Regression risks in existing behavior touched by the change.

## Test quality checks

Flag tests that are:

- Only happy-path.
- Tightly coupled to implementation details.
- Overly mocked (mocking the thing under test, or every collaborator).
- Missing realistic data setup.
- Using brittle selectors.
- Asserting something trivially true instead of the actual business behavior.
- Not aligned with product rules in `docs/PRODUCT.md` or conventions in `docs/ARCHITECTURE.md`.

## Vertical-slice checks

For each reviewed feature, check whether tests cover, where the feature has that layer:

- frontend behavior
- API/domain behavior
- persistence behavior
- cross-layer integration (and e2e, if the project has an e2e suite)

Don't require every layer for every story — only the ones the change actually touches.

## Output

Return test findings only, most-severe first, using the severity scale of `firefly:project-review-methodology` (Critical / High / Medium / Low).

For each finding:

- **Severity:**
- **File/location:** `path:line` of the code or test
- **Issue:**
- **Missing scenario:** the concrete case that isn't covered
- **Why it matters:**
- **Recommended fix:** which test, at which level

If coverage in scope is adequate, say so explicitly.
