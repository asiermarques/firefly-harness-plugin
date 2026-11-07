---
name: github-project-tracking
description: Shared GitHub Projects mechanics for the workflow skills — issue and milestone naming conventions, `gh` preflight, project selection, publishing a plan as one milestone per feature with one issue per user story, locating the issue for a task, and moving its Status (In Progress / Done). Invoked by `create-implementation-plan` and `implement-task`; not meant to be run directly.
user-invocable: false
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Bash(mktemp *)
  - Bash(gh auth status *)
  - Bash(gh repo view *)
  - Bash(gh label list *)
  - Bash(gh issue list *)
  - Bash(gh issue view *)
  - Bash(gh issue create *)
  - Bash(gh issue edit *)
  - Bash(gh issue comment *)
  - Bash(gh project list *)
  - Bash(gh project view *)
  - Bash(gh project create *)
  - Bash(gh project field-list *)
  - Bash(gh project item-list *)
  - Bash(gh project item-add *)
  - Bash(gh project item-edit *)
---

# GitHub Project tracking

The single source of truth for how the workflow skills talk to GitHub Issues and GitHub Projects. The calling skill decides **when** something happens (e.g. "publish now", "move to Done only on green"). This skill defines **how**. Follow only the operations the caller asks for.

## Conventions (the contract between skills)

Other skills rely on these to find each other's issues. Never deviate.

- **The milestone is the plan.** One milestone per feature, titled `<id> <Feature name>` (e.g. `001 Readable share link format`). Its description is the plan's Summary. There is no separate plan issue.
- `<id>` and the feature name come from `.workflow/requisites/<id>-<slug>.md` and the plan. Never re-derive them. The feature name is in sentence case.
- **One issue per user story**, titled `US-NNN: <title>` (e.g. `US-001: Record a Person in a new snapshot`), assigned to the feature milestone. `US-NNN` restarts in each feature, so a story is identified by **milestone + `US-NNN`**, never by title alone.
- Story issues carry the `enhancement` label if it exists in the repo. Don't create labels.
- Story issue body: starts with a bold header, then the sections of the user-story template (User story, Production deliverable, Scope, Out of scope / Dependencies, Acceptance criteria, Risks / Open questions / Implementation notes), skipping empty ones:
  ```md
  **Feature:** <id> <Feature name> · **Slice:** <slice name>
  **Decision:** `docs/adr/<NNNN>-<title>.md`
  ```
  The `**Decision:**` line only appears when the story depends on an ADR. Don't put `.workflow/` paths in issue bodies.
- Local files that have an issue carry a header line `GitHub issue: <url>`. The local plan carries `GitHub milestone: <url>`.
- Don't create custom project fields unless the user asks.
- Never close or reopen issues or milestones. Closing issues is left to the PR that merges the work (`Closes #123`). Closing the milestone is left to the user.

## Operation: Preflight

Run before any other operation.

1. `gh auth status`. If `gh` isn't installed or not authenticated, tell the user to run `! gh auth login`.
2. Projects need the `project` scope. Check the `Token scopes:` line of the same output; if `project` is missing, tell the user to run `! gh auth refresh -s project` — don't wait for a later command to fail.
3. `gh repo view --json nameWithOwner,owner`. If the directory isn't a GitHub repo, report it. Keep `<owner>` and `<repo>` for the other operations.

On failure, report it to the caller. The caller decides whether that's blocking (publishing) or not (status tracking).

## Operation: Select project

Used when the caller needs a target project and there isn't one yet.

1. `gh project list --owner <owner> --format json`.
2. Ask with one `AskUserQuestion` call (header `Project`): up to 3 most relevant projects as options, plus "Create a new project". The user can type a project number via "Other".
3. To create one: `gh project create --owner <owner> --title "<feature name>" --format json` and use the returned number.
4. Return `<owner>`, `<repo>`, `<project-number>`.

## Operation: Publish plan

Input from the caller: the plan (for the milestone), one body per user story following the conventions above, the project, and whether local files also exist.

1. **Milestone**: find it by `<id>` (idempotent re-runs):
   `gh api "repos/<owner>/<repo>/milestones?state=all" --paginate --jq '.[] | select(.title | startswith("<id> ")) | {number, title, html_url}'`
   Write the plan Summary to a temp file first (`mktemp -d`) and pass it with `-F description=@<file>` — never inline multi-line text in the shell command. If none exists, create it: `gh api repos/<owner>/<repo>/milestones -f title="<id> <Feature name>" -F description=@<file>`. If it exists, update the description with `gh api -X PATCH repos/<owner>/<repo>/milestones/<number> -F description=@<file>`. If several match, ask the user which one to use. Keep the milestone **title** and **number** for the next steps.
2. **Find existing story issues in that milestone**: `gh issue list --repo <owner>/<repo> --state all --milestone "<milestone title>" --json number,title,url --limit 100`. Match each story by its `US-NNN:` title prefix.
3. **Label**: check once whether `enhancement` exists (`gh label list --repo <owner>/<repo> --search enhancement --json name`). Add `--label enhancement` below only if it does.
4. **Story issues**: write each body to a temp file (`mktemp -d`, never under `.workflow/`). Create with `gh issue create --repo <owner>/<repo> --title "US-NNN: <title>" --body-file <file> --milestone "<milestone title>" [--label enhancement]`, or update a match with `gh issue edit <number> --body-file <file>`.
5. **Add to the project**: for every story issue, `gh project item-add <project-number> --owner <owner> --url <issue-url>`. Re-adding an existing item is harmless.
6. **Cross-link local files** (only if local files exist): add `GitHub issue: <url>` to each `US-NNN.md`, `GitHub milestone: <url>` to the plan, and `#<number>` to each story line in the local plan index.
7. If a `gh` command fails mid-way, stop and report what was already created/updated (milestone and issues, with URLs). Don't retry blindly.

Return: project URL, milestone URL, and created/updated issue counts.

## Operation: Locate task issue

Input: the task argument and, if known, the local task file.

Try in order and stop at the first hit:

1. The argument is a GitHub issue URL or `#<number>` → that issue.
2. The local task file `.workflow/tasks/<id>-<slug>/US-NNN.md` has a `GitHub issue: <url>` line → that issue.
3. The task names a user story and a feature (`<id>`, slug or name) → find the feature milestone (by `<id> ` title prefix, see Publish plan step 1), then the story inside it:
   `gh issue list --repo <owner>/<repo> --state all --milestone "<milestone title>" --search "\"US-NNN:\" in:title" --json number,title,url,state`
   A `US-NNN` without a feature is ambiguous (every feature has a US-001): ask which feature.
4. Otherwise the task is **not tracked**. Don't search GitHub by free text.

For several stories in one run, locate one issue per story.

Then resolve the project and Status options for each issue:

- `gh issue view <n> --json projectItems,state,url` lists its projects and current status.
- Get each project's number with `gh project list --owner <owner> --format json`, matching by title.
- `gh project field-list <number> --owner <owner> --format json` lists the Status options. Map them ignoring case, spaces and hyphens: `In progress`, `in-progress`, `Doing` → **In Progress**. `Done`, `Completed`, `Closed` → **Done**.

### When to ask

Proceed **without asking** when it's unambiguous: the issue came from step 1 or 2 (or an exact title match in step 3), it's in exactly one project, and both statuses map cleanly. Announce it in one line (e.g. "Tracking #123 in project _Firefly roadmap_").

Otherwise ask with one `AskUserQuestion` call (header `GitHub`):

- several candidate issues → which one(s)?
- the issue is in more than one project → which project(s)?
- no Status option clearly maps → which option to use?
- the issue is already Done or closed → move it back to In Progress, or leave it alone?

Always include "Don't update GitHub" as an option. If chosen, return **not tracked** for the rest of the run.

## Operation: Set status

Input: issue URL, project number, owner, target status (In Progress or Done, as mapped above).

```bash
gh project item-edit <number> --owner <owner> --url <issue-url> --field "Status" --value "<mapped option>"
```

If the installed `gh` doesn't support `--url`/`--field`/`--value`, fall back to node IDs: project id from `gh project view <number> --owner <owner> --format json`, item id from `gh project item-list <number> --owner <owner> --format json` (match `content.url`), and field/option ids from `field-list`. Pass them as `--id`, `--project-id`, `--field-id`, `--single-select-option-id`.

## Operation: Record completion

Input: story issue(s), a short completion note (what shipped, test files that prove it, scope deviations).

1. Comment on each story issue: `gh issue comment <n> --body-file <file>`.
2. Check the milestone's progress: `gh api repos/<owner>/<repo>/milestones/<number> --jq '{open_issues, closed_issues}'` plus the Status of its issues in the project. If every story is Done, tell the user the milestone is ready to be closed. Don't close it yourself.
