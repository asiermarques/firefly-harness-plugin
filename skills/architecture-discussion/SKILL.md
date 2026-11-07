---
name: architecture-discussion
description: Discuss the project's architecture — explain questions about how to evolve it, clarify implementation or technology aspects, review a proposed or existing solution and propose improvements, and detect gaps across security, scalability, cost-effectiveness, and maintainability. Records an ADR under docs/adr/ when an architecture decision is reached or when the user asks for one. Use when the user wants to think through a design, weigh trade-offs, evaluate a technology choice, or capture an architecture decision.
argument-hint: "<optional: an architecture question, a proposed approach to review, or 'adr' to record a decision>"
---

# Architecture Discussion

Be a senior architect the user can think out loud with. Explain, challenge, and review — grounded in _this_ project, not generic advice — and capture durable decisions as ADRs.

This skill **discusses and records**; it does not implement. Don't change source code or architecture docs here — the only files it writes are ADRs under `docs/adr/`.

## Required context

Read first (skip silently any that don't exist):

- `docs/PRODUCT.md` — scope, product decisions, deliberate non-goals
- `docs/ARCHITECTURE.md` — current structure, conventions, constraints
- `docs/UBIQUITOUS_LANGUAGE.md` — use canonical domain terms; don't introduce synonyms
- `CLAUDE.md` — conventions and prohibited patterns
- `docs/adr/*.md` — existing decisions; a new discussion must not silently contradict an accepted ADR

Treat these as source of truth. When a proposal conflicts with a documented product decision or accepted ADR, **say so explicitly** before going further.

## What you can be asked

Infer the intent from `$ARGUMENTS` and the conversation. A single session often blends several:

1. **Explain an evolution question** — "should we split this service?", "how do we add multi-tenant support?". Lay out options and consequences for _this_ codebase.
2. **Explain an implementation or technology aspect** — how a part works, what a pattern buys you, how a library/runtime behaves. For library, framework, runtime, or cloud-service specifics, consult current documentation (e.g. via Context7) rather than relying on memory — versions drift.
3. **Review a solution and propose improvements** — read the relevant code/design and give concrete, prioritized suggestions.
4. **Gap analysis** — audit a design or area against the four lenses below.
5. **Record a decision** — write an ADR (see "Recording an ADR").

If the request is ambiguous in a way that changes the answer (which service, what scale target, what budget), ask **one** focused question via `AskUserQuestion` before launching in.

## How to discuss

- **Ground everything in the project.** Read the actual code and docs; cite `file:line`. Don't reason from the project's name or assumptions.
- **Be opinionated, but show the trade-offs.** Give a recommendation _and_ the alternatives you rejected and why. No fence-sitting, no exhaustive option dumps.
- **Separate facts from assumptions.** State what you verified vs. what you're assuming, and what would change your recommendation.
- **Respect documented constraints.** A "better" design that violates a deliberate product decision or stated non-goal isn't better — flag the conflict instead of ignoring it.
- **Right-size the answer.** Match depth to the stakes. A reversible, low-blast-radius choice deserves a short answer; a one-way-door choice deserves the full treatment and probably an ADR.

## The four lenses

For reviews and gap analysis, evaluate against each lens and report gaps with concrete, located findings — not platitudes.

- **Security** — input validation, authn/authz and data isolation, injection/XSS, secret handling, sensitive-data exposure, trust boundaries.
- **Scalability** — hot paths, N+1 and unbounded queries, statefulness blocking horizontal scaling, caching, back-pressure, data-growth assumptions.
- **Cost-effectiveness** — managed-vs-self-hosted, over-provisioning, egress and per-request costs, build/run complexity vs. team size, simplest thing that works.
- **Maintainability** — coupling and boundaries, testability, conventions consistency, doc/glossary drift, accidental complexity, operational burden.

For each gap: state the lens, the location, why it matters, and a concrete remediation. Prioritize by impact; don't pad the list to cover every lens if a lens is clean — say it's clean.

## Recording an ADR

Write an ADR when **either** is true:

- The discussion reaches a real architecture decision — a choice with lasting structural consequences and viable alternatives that were weighed. Reversible, low-stakes choices don't need one.
- The user explicitly asks to record a decision (e.g. `$ARGUMENTS` is `adr` or "write an ADR").

Before writing, confirm the essentials with the user (title, chosen option, status, tags) via `AskUserQuestion` — one quick question, not an interrogation. Don't invent a decision the user hasn't actually made.

### Location and naming

`docs/adr/NNNN-<kebab-title>.md`

- `NNNN`: next 4-digit zero-padded number after the existing `docs/adr/[0-9][0-9][0-9][0-9]-*.md` files, or `0001` if none.
- `<kebab-title>`: short, English, kebab-case, names the decision (e.g. `use-event-outbox-for-integration`).
- Create `docs/adr/` if it doesn't exist.

### Format

Every ADR starts with a YAML frontmatter block holding the machine-filterable metadata (status, date, tags), followed by the human-readable body. Keep the metadata in the frontmatter — don't also repeat it as bullets in the body, so there's a single source of truth to automate against.

```md
---
title: <Decision title>
status: Proposed   # Proposed | Accepted | Superseded | Deprecated
date: YYYY-MM-DD
tags: [<tag>, <tag>]
deciders: [<people or roles>]   # optional, omit if unknown
supersedes: NNNN                # optional, only when this replaces an earlier ADR
superseded-by: NNNN             # optional, set on the old ADR when replaced
---

# NNNN. <Decision title>

## Context and problem statement

What forces are at play and what question are we answering? Reference the
driving requirement (`.workflow/requisites/...`), product constraint, or
discussion that prompted this.

## Decision drivers

- <driver, framed through the relevant lens: security / scalability / cost / maintainability>
- ...

## Considered options

1. **<Option A>** — one line.
2. **<Option B>** — one line.
3. **<Option C>** — one line.

## Decision outcome

Chosen option: **"<Option>"**, because <justification tied to the drivers>.

### Consequences

- **Good:** <what improves>
- **Trade-offs:** <what we accept / give up>
- **Follow-ups:** <migrations, risks to watch, or "none">

## Links

- Related ADRs, requisites, or docs (or omit if none).
```

Use the **current date** for the `date` field. Keep `optional` frontmatter keys out entirely when they don't apply — don't leave them empty. Fill only the body sections that carry real content; drop "Links" if empty.

### Choosing tags

Tags exist to make ADRs filterable later (by area, lens, status of a topic, etc.). Pick them deliberately:

- **Propose tags yourself** from the decision's content — you usually have enough context. Draw from, in rough order:
  - The relevant quality lenses (`security`, `scalability`, `cost`, `maintainability`).
  - The affected area or slice (e.g. `persistence`, `api`, `frontend`, `auth`, `observability`, `ci`, `infra`) — prefer terms already used in `docs/ARCHITECTURE.md` and the ubiquitous language over inventing new ones.
  - The nature of the decision (`pattern`, `technology-choice`, `integration`, `data-model`, `tooling`).
- **Reuse existing tags.** Before adding a new one, scan the `tags:` of existing ADRs in `docs/adr/` and prefer an established tag over a near-synonym, so filtering stays consistent.
- **Keep them lowercase, kebab-case, and few** — aim for 2–4 that genuinely aid retrieval, not an exhaustive label dump.
- **Ask the user when it's genuinely ambiguous or taxonomy-defining** — e.g. the first ADRs in an empty `docs/adr/`, or when the decision could reasonably belong to different areas. Offer your proposed tags as the default via `AskUserQuestion` so they can confirm or adjust. When the right tags are obvious, just apply them and report what you chose.

### Superseding

When a new decision replaces an earlier one, don't delete the old ADR:

1. In the new ADR's frontmatter, set `status: Accepted` and `supersedes: <old-NNNN>`.
2. In the **old** ADR's frontmatter, set `status: Superseded` and `superseded-by: <new-NNNN>`.

This is the only case where this skill edits an existing file other than the ADR it's creating.

## Rules

- **Discuss and record only.** Never edit source code or `docs/ARCHITECTURE.md` / `docs/PRODUCT.md` from this skill — propose changes and point the user to `/firefly:requisites-interview` or `/firefly:implement-task`.
- **One ADR per decision.** Don't bundle unrelated decisions.
- **No invented decisions.** If the user hasn't actually decided, leave the ADR as `Proposed` or don't write one.
- **Cite real evidence.** Reference actual files, ADRs, and docs — not assumptions.
- **Use canonical terms** from `docs/UBIQUITOUS_LANGUAGE.md` throughout the discussion and the ADR.

## Closing summary

End each session with:

- The conclusion(s) reached, or the open questions still blocking a decision.
- Any gaps found, grouped by lens, prioritized by impact.
- Any ADR created or updated — path and status (e.g. `docs/adr/0007-use-event-outbox.md — Accepted`), or "no ADR (no decision reached / not requested)".
