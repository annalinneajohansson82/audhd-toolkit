# Issue tracker: GitHub

Issues live as GitHub issues on `annalinneajohansson82/audhd-toolkit`. All
issue work happens there. The `gh` CLI is the documented mechanism (Hermes has
it, and the engineering skills invoke these commands); an agent without `gh`
uses the equivalent GitHub MCP or API operations — the outcome is what matters,
not the tool.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments`
- **Comment**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..." / --remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

## Pull requests as a request surface

**PRs as a request surface: no.** External PRs are not triaged as feature
requests on this repo — ideas and requests become issues.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Read the GitHub issue, including its comments (`gh issue view <number> --comments`).

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as
tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Destination /
  Notes / Decisions-so-far / Not-yet-specified / Out-of-scope body.
  `gh issue create --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api`
  on the sub-issues endpoint). Where sub-issues aren't enabled, add the child
  to a task list in the map body and put `Part of #<map>` at the top of the
  child body. Labels: `wayfinder:<type>` (`research` / `prototype` /
  `grilling` / `task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies** — the canonical,
  UI-visible representation. Add an edge with
  `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`,
  where `<blocker-db-id>` is the blocker's numeric **database id**
  (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _not_ the `#number` or
  `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open
  blockers only — the live gate). Where dependencies aren't available, fall
  back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A
  ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh issue list --state
  open`, scoped to the map's sub-issues / task list), drop any with an open
  blocker or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --add-assignee @me` — the session's first write.
- **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close
  <n>`, then append a context pointer to the map's Decisions-so-far.
