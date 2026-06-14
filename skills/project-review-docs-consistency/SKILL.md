---
name: project-review-docs-consistency
description: Check whether docs/PRODUCT.md, docs/ARCHITECTURE.md, README files, and inline comments accurately reflect the current implementation.
user-invocable: false
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Project docs consistency review

Use this skill to find documentation drift between the current implementation and project documentation.

## Required docs

Read:

- `docs/PRODUCT.md`
- `docs/ARCHITECTURE.md`
- `README.md`

Also inspect README or setup docs if relevant.

## Review checks

Compare documentation against the codebase.

Check whether `docs/PRODUCT.md` accurately describes:

- implemented features
- planned features
- user flows
- product constraints
- business rules
- deliberate exclusions
- metrics behavior
- the decision that metrics use raw counts only, with no goals or targets
- incorrect or outdated directory or filename references

Check whether `docs/ARCHITECTURE.md` accurately describes:

- current folder structure
- vertical slice boundaries
- frontend/backend split
- API patterns
- database / data-access usage
- commands
- aliases
- test conventions
- styling / design-system setup
- integration points
- known constraints

Check other docs for:

- outdated setup commands
- incorrect paths
- stale examples
- references to removed files
- comments that describe old behavior
- mismatch between planned and implemented state

## Styling / design-system check

Flag documentation or code that assumes an outdated styling or design-system configuration when the project has since changed how styles or design tokens are defined.

## Output

Return documentation findings only.

For each finding, include:

- Severity:
- File/location:
- Current documentation:
- Actual implementation:
- Why it matters:
- Recommended fix:

Do not update documentation unless explicitly asked.
