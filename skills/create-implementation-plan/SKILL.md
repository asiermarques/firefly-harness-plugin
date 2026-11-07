---
name: create-implementation-plan
description: Turn a `.workflow/requisites/<id>-<slug>.md` file into a vertical-slice implementation plan plus one user story per slice. Reads PRODUCT.md, ARCHITECTURE.md, UBIQUITOUS_LANGUAGE.md, and the requisites file; asks whether to write the plan locally (`.workflow/implementation-plans/<id>-<slug>.md` + `.workflow/tasks/<id>-<slug>/US-XXX.md`), publish it to a GitHub Project as issues, or both. Use AFTER the requisites interview, BEFORE implementation.
argument-hint: "<requisites file path, feature slug, ID, or feature name>"
disable-model-invocation: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(mktemp *)
  - AskUserQuestion
  - Skill
---

# Create implementation plan

Plan the feature `$ARGUMENTS` into vertical slices and per-slice user stories. The plan is an index; the user-story files carry the detail.

## Process (in order)

1. **Resolve the requisites file** (see "Resolving the input"). If none exists, stop and tell the user to run `/firefly:requisites-interview` first.
2. Read `./docs/PRODUCT.md`, `./docs/ARCHITECTURE.md`, the requisites file, and `./docs/UBIQUITOUS_LANGUAGE.md` if present. If PRODUCT.md or ARCHITECTURE.md is missing, stop and report.
3. **Readiness gate**: if requisites status is `Not ready`, stop and report the missing items. If `Ready with assumptions`, proceed but echo the assumptions in the plan Summary.
4. **Choose the output destination** (see "Output destination"). Always ask — never assume a default. If GitHub is involved, set it up now so it fails before any planning work.
5. **Glossary check**: write the plan and stories using canonical terms from `UBIQUITOUS_LANGUAGE.md`. If the requisites file already uses canonical terms, no extra work. If you spot a new/conflicting term, flag it in the final response — don't rewrite the glossary here.
6. Inspect code only when needed to confirm existing patterns the plan will reuse. Don't survey the whole codebase.
7. Split into vertical slices (one slice = one independently releasable increment). Split each slice into user stories that are each independently deliverable.
8. Derive paths and titles (see "Naming").
9. Produce the output for the chosen destination: write the local files (see "Plan output" / "User story output") and/or publish to GitHub (see "Publishing to GitHub"). Skip sections that don't apply — don't fill with "Not applicable".
10. Final response: just locations and counts (see "Final response").

## Output destination

Ask with a single `AskUserQuestion` call (single-select, header `Destination`):

- Question: "Where should the implementation plan and user stories for `<id>-<slug>` be created?"
- Options (exactly these three):
  1. **Local markdown files** — write the plan and `US-XXX.md` files under `.workflow/`.
  2. **GitHub Project** — create a milestone for the feature (it plays the role of the plan) and one GitHub issue per user story assigned to it, and add the issues to a GitHub Project. No local files.
  3. **Both** — write the local `.workflow/` files and also publish them to a GitHub Project, cross-linking each file with its issue.

Don't proceed until the user has picked one. If they answer "Other", map it to one of the three options or ask again.

### GitHub setup (options 2 and 3)

**Invoke the `firefly:github-project-tracking` skill now**, before continuing. Don't improvise `gh` commands. From it, run **Preflight**, then **Select project**. If Preflight fails, stop: publishing can't proceed. Keep `<owner>`, `<repo>` and `<project-number>` for step 9.

## Resolving the input

`$ARGUMENTS` may be a full path, a file name, a slug-with-ID, a slug-without-ID, an ID alone, or a free-form feature name.

Match order:
1. If it's an existing path under `.workflow/requisites/`, use it.
2. Glob `.workflow/requisites/*.md` and match by exact filename, then by `<id>-*`, then by `*-<slug>.md`.
3. If multiple match, ask via `AskUserQuestion` which file to use.
4. If none match, stop. Do not invent requirements or create a stub requisites file.

The plan and tasks **must reuse the same `<id>-<slug>` as the requisites file** — never re-derive.

## Naming

Given requisites file `.workflow/requisites/<id>-<slug>.md`:

- Plan: `.workflow/implementation-plans/<id>-<slug>.md`
- Tasks dir: `.workflow/tasks/<id>-<slug>/`
- Task files: `.workflow/tasks/<id>-<slug>/US-001.md`, `US-002.md`, …

GitHub issue titles (options 2 and 3) follow the **Conventions** in `firefly:github-project-tracking`.

When updating an existing plan, keep `US-XXX` IDs and statuses stable. Don't renumber. On GitHub, update the existing issues instead of creating duplicates.

## Rules

- Don't write code. Don't modify production files.
- Don't invent unresolved requirements. If a gap blocks planning, surface it as an open question and lower readiness.
- Plan by **vertical slice**, not by technical layer. No separate frontend/backend/infra/observability/testing sections.
- Testing is implicit per user story — the implementation skill handles it.
- Keep the plan an **index**. Story detail lives in `US-XXX.md`.
- Use canonical glossary terms throughout.
- Skip sections with no real content. Never write "Not applicable".

## Plan output

Write `.workflow/implementation-plans/<id>-<slug>.md` (local). On GitHub, its Summary becomes the milestone description. Required: header, Sources, Summary, Vertical slices, Implementation readiness. Everything else is optional.

```md
# Implementation plan: <feature name>

Requirement ID: <id>
Feature slug: `<slug>`
Plan path: `.workflow/implementation-plans/<id>-<slug>.md`
Tasks dir: `.workflow/tasks/<id>-<slug>/`

## Sources

- Requirements: `.workflow/requisites/<id>-<slug>.md`
- Product: `./docs/PRODUCT.md`
- Architecture: `./docs/ARCHITECTURE.md`
- Glossary: `./docs/UBIQUITOUS_LANGUAGE.md` (if present)

## Summary

2–5 sentences. What ships, how it's split, key constraints, key risks. Echo any "Ready with assumptions" items from the requisites file.

## Vertical slices

### Slice 1: <name>

Status: Not started
Goal: <one sentence>
Scope: <bullets>
Out of scope: <bullets, omit if empty>
Dependencies: <bullets, omit if none>

#### Rollout (optional)

Only include lines that matter for this slice (feature flag, migration, rollback). Omit the section if it's all defaults.

#### User stories

- [ ] US-001: <title> — `.workflow/tasks/<id>-<slug>/US-001.md` — <one-sentence deliverable>
- [ ] US-002: <title> — `.workflow/tasks/<id>-<slug>/US-002.md` — <one-sentence deliverable>

#### Risks / Open questions (optional)

RISK-001 / OQ-001 if ≥3 items. Otherwise plain bullets. Skip if none.

---

### Slice 2: <name>

Same structure.

## Cross-slice dependencies (optional)

Skip if none.

## Global risks / open questions (optional)

Skip if none.

## Implementation readiness

`Ready` | `Ready with assumptions` | `Not ready` — one sentence justifying.
```

## User story output

Write `.workflow/tasks/<id>-<slug>/US-XXX.md` (local). On GitHub, the same sections form the story issue body. Required: header, User story, Production deliverable, Acceptance criteria. Everything else is optional.

````md
# US-XXX: <title>

Status: Not started
Feature: `<id>-<slug>`
Slice: `<slice name>`
Plan: `.workflow/implementation-plans/<id>-<slug>.md`
Requirements: `.workflow/requisites/<id>-<slug>.md`

## User story

As a <actor>, I want <capability>, so that <benefit>.

## Production deliverable

One paragraph. What is releasable after this story merges, and how to demo it.

## Scope (optional)

Bullets covering only the categories that apply. Omit the rest — don't list "Frontend: Not applicable".

## Out of scope / Dependencies (optional)

Skip if empty.

## Acceptance criteria

```gherkin
Given ...
When ...
Then ...
```

Add scenarios until the main paths and one or two edge cases are covered (usually 3–6).

## Risks / Open questions / Implementation notes (optional)

Include only if they carry information the implementer can't derive from the requisites + architecture docs.
````

## Publishing to GitHub

Only for options 2 (GitHub Project) and 3 (Both). Build the milestone description from the plan Summary and each story issue body from the "User story output" sections, using the GitHub header and title conventions of `firefly:github-project-tracking` (no `.workflow/` paths). Write the bodies to a temp dir (`mktemp -d`), never under `.workflow/`. In option 3, write the local files first.

Then run the **Publish plan** operation of `firefly:github-project-tracking` (already loaded in step 4) with the milestone description, the story bodies and the selected project. In option 3, that operation also cross-links the local files with their issues and milestone.

In option 2, plan-level content beyond the Summary (cross-slice dependencies, global risks/open questions) has no home on GitHub: put it in the stories it affects, and list anything left over in the final response.

## Quality checklist (run before reporting done)

- The user picked the destination via `AskUserQuestion`; the output matches that choice.
- Requisites file exists and was the source.
- Plan path / issue titles use the same `<id>-<slug>` as the requisites file, and issue titles follow the `github-project-tracking` conventions.
- Local: every `US-XXX` in the plan has a matching file under `.workflow/tasks/<id>-<slug>/`.
- GitHub: the feature milestone exists with the plan Summary as description; every `US-XXX` has exactly one issue in that milestone, and all of them are in the chosen project.
- Both: every local file links its issue, and vice versa.
- No technical-layer groupings, no testing section.
- No "Not applicable" fillers.
- Canonical glossary terms used throughout.

## Final response

Return only:
- destination (local / GitHub Project / both)
- plan path and tasks dir (if local)
- GitHub project URL and milestone URL, plus created/updated issue counts (if GitHub)
- requisites path
- slices count / user stories count
- readiness
- open questions count
- `glossary update needed: <terms>` if step 5 surfaced any new/conflicting terms
- critical blockers, if any
