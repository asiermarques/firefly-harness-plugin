# Firefly Harness

A [Claude Code](https://claude.com/claude-code) plugin that bundles a set of **skills** for a disciplined, spec-driven development workflow: capture requirements, build a shared domain language, turn requirements into a vertical-slice plan, implement each task with strict TDD, and review the result against the project's product, architecture, security, test, and documentation standards.

The skills are deliberately stack-agnostic — they encode a *methodology* (DDD glossary → requisites → plan → TDD implementation → review), not a specific language or framework.

## The workflow

```
ubiquitous-language ──► requisites-interview ──► create-implementation-plan ──► implement-task ──► (review)
   (domain glossary)      (.workflow/requisites)   (.workflow/implementation-      (TDD + docs       project-review-*
                                                     plans + tasks/US-XXX)          update)
```

## What's inside

### Workflow skills (you invoke these explicitly)

| Skill | What it does |
| --- | --- |
| `ubiquitous-language` | Extracts a DDD-style glossary of domain terms, flags ambiguities, proposes canonical names, and vets new terms against the existing glossary. Saves to `docs/UBIQUITOUS_LANGUAGE.md`. |
| `requisites-interview` | Captures requirements for a new feature: reads the product/architecture/glossary docs, surfaces conflicts, asks closed clarifying questions, and writes `.workflow/requisites/<id>-<slug>.md`. |
| `create-implementation-plan` | Turns a requisites file into a vertical-slice implementation plan plus one user-story file per slice under `.workflow/implementation-plans/` and `.workflow/tasks/`. |
| `implement-task` | Implements a single task end-to-end with strict TDD (red → green → refactor), runs the full test + e2e suites, and updates the workflow files and docs. |
| `architecture-discussion` | Discusses how to evolve the architecture, explains implementation/technology aspects, reviews a solution and proposes improvements, and detects gaps across security, scalability, cost-effectiveness, and maintainability. Records an ADR under `docs/adr/` when a decision is reached or when you ask for one. |


### Review skills (invoked automatically by Claude during a review)

| Skill | What it does |
| --- | --- |
| `project-review-methodology` | Orchestrates a structured review across code quality, tests, performance, product alignment, security, and docs, then returns prioritized findings. |
| `project-review-security` | Reviews input validation, multi-user data isolation, injection/XSS risks, and sensitive-data exposure. |
| `project-review-testing` | Reviews test coverage and quality: edge cases, error paths, integration scenarios, vertical-slice completeness. |
| `project-review-docs-consistency` | Checks that `docs/PRODUCT.md`, `docs/ARCHITECTURE.md`, READMEs, and comments still match the implementation. |

These skills set `user-invocable: false` — you don't call them directly; the review methodology (or Claude) pulls them in when relevant.



## Expected project layout

The skills read from and write to a conventional structure. They degrade gracefully when files are missing, but you get the most out of the plugin when the target project has:

```
your-project/
├── CLAUDE.md                      # conventions, layering, prohibited patterns
├── docs/
│   ├── PRODUCT.md                 # scope, features, product decisions
│   ├── ARCHITECTURE.md            # folder structure, slice boundaries, conventions
│   ├── UBIQUITOUS_LANGUAGE.md     # canonical domain glossary
│   ├── DESIGN.md                  # design system / tokens (for UI work)
│   └── adr/                       # architecture decision records (NNNN-title.md)
└── .workflow/
    ├── requisites/                # requisites-interview output
    ├── implementation-plans/      # create-implementation-plan output
    └── tasks/<id>-<slug>/US-XXX.md
```

## Project structure

```
firefly-harness-plugin/
├── .claude-plugin/
│   └── plugin.json                # Plugin manifest (name, version, metadata)
├── commands/
│   └── harness-check.md           # /harness-check slash command
├── skills/                        # One folder with a SKILL.md per skill
│   ├── ubiquitous-language/
│   ├── requisites-interview/
│   ├── create-implementation-plan/
│   ├── implement-task/
│   ├── architecture-discussion/
│   ├── project-review-methodology/
│   ├── project-review-security/
│   ├── project-review-testing/
│   └── project-review-docs-consistency/
└── README.md
```

## Installation

### Option A — Local marketplace (recommended for development)

A plugin is installed through a marketplace. The quickest path while developing is to point Claude Code at a local marketplace that lists this plugin.

1. Create a marketplace manifest somewhere on your machine, e.g. `~/my-marketplace/.claude-plugin/marketplace.json`:

   ```json
   {
     "name": "local-dev",
     "owner": { "name": "Asier Marques" },
     "plugins": [
       {
         "name": "firefly-harness",
         "source": "/Users/asiermarques/Documents/projects/claude-training/firefly-harness-plugin",
         "description": "Spec-driven development workflow skills for Claude Code."
       }
     ]
   }
   ```

2. Add the marketplace and install the plugin from within Claude Code:

   ```
   /plugin marketplace add ~/my-marketplace
   /plugin install firefly-harness@local-dev
   ```

### Option B — From a Git repository

1. Push this plugin (or a marketplace that references it) to a Git host.
2. In Claude Code:

   ```
   /plugin marketplace add <your-org>/<your-repo>
   /plugin install firefly-harness@<marketplace-name>
   ```

After installing, restart Claude Code (or run `/plugin` and reload) so the new skills and command are picked up.

## Usage

Run the workflow skills in order for a new feature:

1. **Harden terminology** — invoke `ubiquitous-language` to capture or vet the domain terms for the feature.
2. **Capture requirements** — invoke `requisites-interview` with the feature request. It interviews you and writes a requisites file.
3. **Plan** — invoke `create-implementation-plan` with the requisites file. It produces a vertical-slice plan and per-user-story task files.
4. **Implement** — invoke `implement-task` with a task description (or user story). It drives the TDD loop, runs the suites, and updates the docs/workflow files.
5. **Review** — ask Claude to review the changes; it pulls in `project-review-methodology` and the specialized `project-review-*` skills automatically and returns prioritized findings.

Outside this linear flow, invoke `architecture-discussion` whenever you want to think through a design, evaluate a technology choice, weigh trade-offs, or capture an architecture decision as an ADR — before, during, or after the pipeline.

At any point, run `/harness-check` to confirm lint, build, and tests are green.

You can confirm what loaded with `/plugin` (lists installed plugins).

## Customizing

- **Add a skill** — create `skills/<your-skill>/SKILL.md` with `name` and `description` frontmatter. The `description` is what Claude uses to decide when a model-invoked skill applies. Set `disable-model-invocation: true` for skills only you should trigger, or `user-invocable: false` for skills only Claude should pull in.
- **Add a command** — create `commands/<name>.md`; the file body is the prompt, and `$ARGUMENTS` is substituted with whatever the user types after the command.
- **Adapt to your stack** — the skills reference a `docs/` + `.workflow/` layout and a layered backend / vertical-slice convention. Adjust the SKILL.md bodies to match your project's actual conventions.

Bump the `version` in `.claude-plugin/plugin.json` whenever you publish changes.

## License

MIT
