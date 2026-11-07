---
name: implement-task
description: Implement a coding task end-to-end using strict TDD (red → green → refactor), then run the full test and e2e suites and update the workflow files and project documentation. If the task is tracked in a GitHub Project, moves it to In Progress when work starts and to Done when it's finished. Use when the user provides a task description, user story, or GitHub issue and asks for it to be implemented.
argument-hint: "<task description, user story (e.g. 003 US-002), issue URL, or #number>"
disable-model-invocation: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
  - Skill
---

# Implement task

Drive the implementation of a single, well-scoped coding task: `$ARGUMENTS` (e.g. `/firefly:implement-task add an "archive item" action that hides an item from the picker without deleting it`). The skill is responsible for delivering working, tested, documented code — not for inventing scope.

## Inputs

- **The task description** — passed as the skill argument. Treat it as the source of truth for what to build. If it is ambiguous in a way that affects the data model, public API, or user-facing behavior, ask the user **one** clarifying question before writing code. Do not ask about implementation details you can decide yourself.
- The argument may also be a GitHub issue URL, an issue number (`#123`), or a user-story reference with its feature (`<NNN>-<slug> US-002`, `003 US-002`). In that case the story text (local `US-NNN.md` file or issue body via `gh issue view <n> --json title,body`) is the task description.

## GitHub Project tracking

**Before writing the first test, invoke the `firefly:github-project-tracking` skill.** Don't improvise `gh` commands. Use its operations like this:

1. **Preflight**. If it fails (no `gh`, not authenticated, not a GitHub repo, missing `project` scope), don't block the implementation: skip tracking and say so in the summary.
2. **Locate task issue** for the task (or one per user story when several are implemented in one run). It decides whether to proceed silently or ask the user. If the task isn't tracked on GitHub, skip the rest of this section, apart from one line in the summary.
3. **Set status → In Progress** for each story issue.
4. At the end, only after closing-sequence steps 1–4 have passed (full suite green, e2e green, workflow files and docs updated): **Set status → Done** for each story issue, then **Record completion** with what shipped, the test files that prove it, and any scope deviations.

If the task ends **not done** (red tests you couldn't fix, a blocker, or the user stopped you), leave the issue in In Progress and say why in the summary. Never move to Done on red. Never close issues.

## Operating principles

Read these before touching code (skip silently any that don't exist). The project's own conventions always win over the defaults below:

- `CLAUDE.md` — conventions, layering, prohibited patterns, test commands
- `docs/PRODUCT.md` — current scope (is this already implemented? is it explicitly out of scope?) and deliberate product decisions
- `docs/ARCHITECTURE.md` — file layout, slice boundaries, layer conventions
- `docs/DESIGN.md` — design system for any UI touched
- `docs/UBIQUITOUS_LANGUAGE.md` — use the canonical names in code, tests, and copy
- `docs/adr/*.md` — accepted architecture decisions the change must respect

If the task contradicts a documented product decision or an accepted ADR, stop and ask before writing code.

Defaults this workflow assumes, unless the docs above say otherwise:

- **Vertical slices.** A feature ships end-to-end (DB → API → UI) in one go, not layer by layer. Don't stop at the backend.
- **Strict TDD.** Tests are written first, fail first, then code makes them pass. No exceptions — this one is not overridable.
- **Layered backend.** Each slice separates a pure domain layer, an infrastructure layer (the persistence adapter, which owns transactions), a transport layer (e.g. HTTP routing + request-schema validation), and its tests. Use the project's actual folder names and respect those boundaries.
- **Design tokens, not raw values.** If `docs/DESIGN.md` defines tokens, all UI work goes through them — never introduce raw color/utility classes.
- **Product decisions are not oversights.** A deliberate constraint in `docs/PRODUCT.md` (something the product intentionally doesn't do) is respected, not "fixed".

## The TDD loop

For every layer the task touches, apply red → green → refactor:

### 1. Plan the slice

Before writing any code, write down (as a todo list for anything beyond a trivial change):

- Which slice(s) are involved? (as defined in `docs/ARCHITECTURE.md`)
- What's the minimal **observable** behavior change? (What does the user see or what does an API caller get?)
- Which layer needs new types? Which layer needs a new repository method? Which router gets a new route?
- What's the smallest end-to-end test that proves the feature exists? **Start there.**

### 2. Write the first failing test

Always start at the outermost meaningful layer for the change:

- Feature touches the API → write an integration test that exercises the route/handler against the repository.
- Feature is pure domain logic (invariants, validation) → write a domain unit test that constructs values and asserts the invariant function throws/passes.
- Feature is UI-only (visual or interaction) → write a component test using the project's frontend test utilities.

Run the test. **Confirm it fails for the right reason** (assertion fails, not "module not found" buried under other errors). If it fails for the wrong reason, fix that first.

### 3. Make it green

Write the minimum code to pass. Move down the layers as needed, in the order `docs/ARCHITECTURE.md` prescribes — by default:

- Domain types and invariant functions first (pure, sync, throw domain-error types)
- Repository port (an interface defined in the domain layer)
- Persistence adapter in the infrastructure layer (the only place transactions are owned)
- Transport route/handler using the project's request-validation helper (never hand-roll request-body checks)
- Frontend: data-fetching hook for the slice, then the component

Run the test after each meaningful step. Don't stack unverified layers.

### 4. Refactor

Once green, look at what you wrote. Remove duplication, tighten names, delete dead branches. **Don't add features, error handling for impossible cases, or abstractions for hypothetical futures.** Three similar lines beats a premature helper.

Re-run the test. It must still pass.

### 5. Repeat for the next observable behavior

Add the next failing test (edge case, error path, second user-facing variation). Loop until the task is complete.

## When the implementation is complete

**Mandatory closing sequence — do not skip any step.**

### 1. Run the full test suite

Run the project's full test command (see `CLAUDE.md` / `README.md`).

It must run **every workspace/suite in the project** and exit clean. If anything is red:

- A test you didn't write failed → you broke something unrelated. Investigate and fix the root cause; don't disable the test.
- Your own test fails after a refactor → the refactor was wrong; revert or fix.

Don't declare the task done while tests are red.

### 2. Run the end-to-end suite and make it green

Run the project's end-to-end test command.

If the project has **no e2e suite at all**, say so in the summary and move on — don't build e2e infrastructure as a side effect of this task. Otherwise, this step is **not optional**. The unit/integration suites pass with the new code; the e2e suite proves the feature still works through a real browser against a real backend, and — just as importantly — that **the rest of the app still works** after your changes.

Vertical slices routinely break existing e2e flows in ways unit tests can't see:

- A new section on a page can push a previously-visible element below the fold, breaking a selector.
- A new modal can intercept clicks that another test assumed went straight to the page.
- A new query on a page can race with an existing assertion that didn't `await` it.
- A new migration can leave the e2e DB in a state the global setup didn't expect.
- A new localization key added in one language but not another can crash a localized e2e run.

When the e2e suite is red, do **not**:

- Mark the failures as flaky and re-run.
- Skip or silence the failing tests.
- Comment out assertions.
- Declare the task done with a note like "e2e failures unrelated, will fix later."

Instead, for **every failing e2e test**, decide which of these applies and act:

1. **The test is asserting old behavior your change intentionally replaced.** Update the test to assert the new behavior. This is the most common case for UI work. The test stays — only the expectation changes.
2. **The test is asserting general behavior your change accidentally broke.** That's a regression. Fix the production code; do not loosen the assertion.
3. **The feature you just added is e2e-worthy and not yet covered.** Add a new e2e spec alongside the existing ones that exercises the user-visible flow end-to-end (the same one your task description demoed). Follow the patterns and shared helpers already in the e2e suite — don't introduce a parallel setup.

Loop: run the e2e suite, fix or update, re-run. The task is not done until the full e2e suite exits clean — same bar as the unit/integration suite.

If a failure looks like genuine e2e flake (timing, network jitter, headless quirks) and not a behavior change, **first** prove it with a targeted re-run of that spec (repeating it a few times) before adjusting anything. Don't paper over a real failure as flake.

### 3. Update the workflow files (if they exist for this task)

This project uses `.workflow/` for the upstream artifacts that birthed the task:

- `.workflow/requisites/<NNN>-<slug>.md` — the feature requisites
- `.workflow/implementation-plans/<NNN>-<slug>.md` — the implementation plan
- `.workflow/tasks/<NNN>-<slug>/US-NNN.md` — the individual user-story tasks

Before declaring the implementation complete:

1. **Locate the parent feature.** If the task description references a user story (e.g. `US-002`), a feature slug, or a feature number, search `.workflow/tasks/`, `.workflow/requisites/`, and `.workflow/implementation-plans/` for the matching `<NNN>-<slug>`. If the task description doesn't name one, ask the user which feature it belongs to before continuing (or confirm there's no associated workflow artifact).
2. **Update the task file** (`.workflow/tasks/<NNN>-<slug>/US-NNN.md`): mark the user story as completed, note any scope deviations, and record which test files prove it.
3. **Update the implementation plan** (`.workflow/implementation-plans/<NNN>-<slug>.md`): tick off the step(s) the task covered. If the plan was wrong in a way that mattered (a step turned out to be unnecessary, or a new step had to be added), update the plan to reflect what actually happened — the plan is a living document, not a historical record.

If none of these files exist (or the task is unrelated to any feature in `.workflow/`), say so in the summary and skip this step. Don't create workflow files defensively. If the feature only exists on GitHub (no local files), the GitHub issue is the workflow artifact. Its update happens in step 5.

### 4. Update the documentation

When the implementation diverges from the docs, the docs must be updated. Walk each existing doc and update it as needed:

- **`docs/PRODUCT.md`** — if the task added/changed/removed a user-facing feature, update the features section. If the task touched a deliberate constraint, update the product-decisions section.
- **`docs/ARCHITECTURE.md`** — if the task added a new slice, a new layer convention, or a new shared utility worth knowing about, mention it here. Do not narrate routine changes.
- **`docs/UBIQUITOUS_LANGUAGE.md`** — if the task introduced new domain vocabulary (a new entity, archetype, or status), add it here with its canonical name and any aliases to avoid. For anything contentious, recommend `/firefly:ubiquitous-language <term>` instead of deciding alone.
- **`docs/DESIGN.md`** — if the task added new tokens, utilities, components, or rules to the design system, update this too.
- **`README.md`** — usually only if commands or quick-start changed.
- **`docs/adr/`** — don't write ADRs here. If the task forced a real architecture decision, flag it in the summary and suggest `/firefly:architecture-discussion adr`.

If none of the docs need updating, say so explicitly in the final summary — don't quietly skip.

### 5. Move the GitHub issue(s) to Done (if tracked)

Follow step 4 of "GitHub Project tracking". Skip if the task isn't tracked on GitHub or the user chose not to update it.

### 6. Summary to the user

Report back with:

- A one-line description of what shipped
- The result of the full test suite (pass count or "all suites pass")
- The result of the e2e suite (pass count, plus a list of e2e specs that were **added** or **updated** to reflect new behavior)
- Which workflow files were updated (task file, plan, requisites) — or "no associated workflow artifact"
- GitHub tracking: which issues were moved (with URLs and final status), or "not tracked on GitHub" / "skipped: <reason>"
- Which docs were updated (or "no doc changes needed because …")
- Anything explicitly **not** implemented that the task description could have implied — give the user a chance to redirect
- Any **requisites changes** — flag these prominently so the user can validate them

## Hard rules — never violate

- **No code before a failing test.** If you find yourself typing `export function` before there's a red test, stop and write the test.
- **No skipping the full test suite at the end.** Even if the tests you wrote pass in isolation, the full suite catches the rest.
- **No skipping the e2e suite at the end.** When the project has one, a green unit/integration run is not enough — the e2e suite is the final gate. Run it, and if it's red, update broken specs to assert the new behavior (or add new specs for new features) until it's green. UI/UX work that doesn't end with green e2e is not done.
- **No silencing e2e failures.** Don't skip / comment out assertions / loosen selectors to "make it pass." Either fix the production code (regression) or update the test to assert the new intended behavior (changed UX).
- **No raw values where the design system has tokens.** Use the design tokens from `docs/DESIGN.md`.
- **No transactions outside the persistence layer** (or wherever `docs/ARCHITECTURE.md` places them).
- **No hand-rolled request-body checks in routes.** Always go through the project's request-validation helper with a schema.
- **No commits.** Implement and report. Only commit when the user explicitly asks.
- **No silent doc drift.** If you change observable behavior and don't update PRODUCT.md, you didn't finish the task.
- **No silent workflow drift.** If the task came from a `.workflow/` artifact, update it. Don't leave the plan or task file pointing at a past reality.
- **No Done on red.** A GitHub issue only moves to Done after the full suite and e2e suite are green. Never close issues. That's the PR's job.

## Anti-patterns to refuse

- Writing all the code, then writing tests at the end. That's not TDD — that's test-after, and it produces tests that confirm what you already wrote rather than tests that drive design.
- Stubbing out future work with `// TODO` comments. Either implement it now (if it's in scope) or don't reference it (if it's not).
- Adding feature flags, backwards-compatibility shims, or "in case we need it later" parameters.
- Catching and re-throwing errors with no added context. Let them propagate.
- Adding new dependencies for problems the existing stack already solves.
- Declaring the task done with green unit tests but unreviewed e2e. The e2e suite is part of "done," not a follow-up.
- Updating an e2e test to expect _nothing_ (`expect(...).toBeTruthy()` on something trivially true) instead of expecting the new behavior. If the change altered UX, the test should now positively assert the new UX, not just stop failing.
