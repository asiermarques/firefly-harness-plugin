---
name: project-review-methodology
description: Structured project review of recently written or modified code across code quality, tests, performance, product alignment, security, and documentation, producing a prioritized, severity-ranked findings report. Use when the user asks to review changes, a branch, a feature, or the project before merging or shipping.
user-invocable: false
allowed-tools:
  - Read
  - Glob
  - Grep
  - Skill
  - Bash(git status *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git show *)
---

# Project review methodology

Perform a structured review of recently written or modified code and return prioritized, actionable findings. This skill **reviews only** — don't modify code or docs unless the user explicitly asks.

## Required context

Read first (skip silently any that don't exist, but note in the report which ones were missing):

- `CLAUDE.md` — conventions and prohibited patterns
- `docs/PRODUCT.md` — product intent, scope, deliberate product decisions
- `docs/ARCHITECTURE.md` — structure, slice boundaries, conventions, constraints
- `docs/UBIQUITOUS_LANGUAGE.md` — canonical domain terms
- `docs/adr/*.md` — accepted architecture decisions

Treat them as source of truth. A deliberate product decision or accepted ADR is not a bug — don't flag it as one. If you think the decision itself is wrong, say so separately as a question, not as a finding.

## Process

1. **Identify the review scope.**
   - If the user named a target (branch, PR, files, feature), use it.
   - Otherwise use the current changes: `git status` and `git diff` (plus `git diff <base>...HEAD` for a feature branch).
   - If there are no changes and no target, ask which area to review instead of reviewing the whole project.
   - State the scope at the top of the report.
2. **Review across every dimension**, reading the actual code in scope (and enough surrounding code to judge it):
   - code quality and adherence to the documented conventions
   - testing gaps
   - performance
   - product/business alignment (against `docs/PRODUCT.md` and, if present, the related `.workflow/` requisites and plan)
   - security
   - documentation accuracy
3. **Delegate the specialized dimensions** by invoking these skills for the same scope:
   - `firefly:project-review-security`
   - `firefly:project-review-testing`
   - `firefly:project-review-docs-consistency`
4. **Verify every finding** before reporting it: re-read the cited code and confirm the problem is real in this codebase. Drop speculative findings, or mark them explicitly as "needs confirmation".
5. **Merge and deduplicate** findings from all dimensions. When two dimensions report the same root cause, keep one finding and mention both angles.
6. **Classify by severity** (scale below) and sort most-severe first.
7. **Return the report** (format below) to the user or calling agent.

## Severity

This scale is shared by all `project-review-*` skills.

- **Critical**: security issue, data corruption or loss, broken core flow, major product contradiction.
- **High**: significant bug, missing error handling on a real path, large test gap, serious performance issue.
- **Medium**: maintainability issue, moderate test gap, minor behavior mismatch, partial documentation drift.
- **Low**: naming, small cleanup, minor documentation or style improvement.

## Finding format

For each finding:

- **Severity:**
- **Dimension:** quality / tests / performance / product / security / docs
- **File/location:** `path:line`
- **Issue:**
- **Why it matters:**
- **Recommended fix:**

Be specific. Avoid vague advice ("consider improving error handling") — name the exact case and the fix.

## Report

1. Scope reviewed and docs read (or missing).
2. One-line verdict (e.g. "2 High, 3 Medium — not ready to merge until the High items are fixed").
3. Findings, most-severe first.
4. Dimensions with no findings, listed explicitly as clean — don't pad the report to fill every dimension.
5. Open questions (e.g. product decisions that look questionable), if any.

## Checklist

Before finishing, verify:

- The scope is stated and every dimension was considered.
- The specialized review skills were run for the same scope.
- Every finding cites a real location and was verified.
- Findings are deduplicated and sorted by severity.
- Recommendations are actionable.
- Intentional product decisions and accepted ADRs were not flagged as bugs.
