---
name: project-review-docs-consistency
description: Find documentation drift — check whether docs/PRODUCT.md, docs/ARCHITECTURE.md, docs/UBIQUITOUS_LANGUAGE.md, docs/DESIGN.md, ADRs, READMEs, CLAUDE.md, and inline comments still match the current implementation. Use when reviewing changes for doc accuracy, or as the documentation dimension of project-review-methodology.
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

# Project docs consistency review

Find drift between the implementation and the project documentation. Focus on the docs affected by the code in scope (the caller's scope, or the current changes from `git diff` if none is given). This skill **reviews only** — don't update documentation unless explicitly asked.

## Docs to check

Read the ones that exist:

- `docs/PRODUCT.md`
- `docs/ARCHITECTURE.md`
- `docs/UBIQUITOUS_LANGUAGE.md`
- `docs/DESIGN.md`
- `docs/adr/*.md`
- `README.md` and any other README or setup docs
- `CLAUDE.md`

A missing doc that the project clearly needs (e.g. no `docs/ARCHITECTURE.md` in a multi-module codebase) is itself a finding.

## Review checks

Compare each doc against the codebase.

`docs/PRODUCT.md` — does it accurately describe:

- implemented vs. planned features
- user flows
- business rules and product constraints
- deliberate exclusions and product decisions (things the product intentionally does *not* do)
- directory or filename references

`docs/ARCHITECTURE.md` — does it accurately describe:

- current folder structure and slice boundaries
- frontend/backend split
- API patterns
- database / data-access usage
- commands and path aliases
- test conventions
- styling / design-system setup
- integration points and known constraints

`docs/UBIQUITOUS_LANGUAGE.md` — do code, UI copy, and other docs use the canonical terms? Flag aliases-to-avoid that appear in new code, and domain concepts in code that are missing from the glossary.

`docs/DESIGN.md` — does it match how styles and design tokens are actually defined? Flag docs or code that assume an outdated styling configuration.

ADRs — does the implementation contradict an `Accepted` ADR without a superseding one?

Other docs and comments:

- outdated setup commands
- incorrect paths, references to removed files
- stale examples
- comments that describe old behavior
- mismatch between planned and implemented state (including `.workflow/` plans and task statuses, if present)

## Output

Return documentation findings only, most-severe first, using the severity scale of `firefly:project-review-methodology` (Critical / High / Medium / Low).

For each finding:

- **Severity:**
- **File/location:** `path:line` of the doc
- **Current documentation:**
- **Actual implementation:** with the `path:line` that proves it
- **Why it matters:**
- **Recommended fix:** the concrete wording or section to change

If the docs in scope are consistent, say so explicitly.
