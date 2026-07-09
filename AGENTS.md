## Agent skills

### Issue tracker

Issues live in GitHub Issues on this repo. External PRs are not a triage surface. See `docs/agents/issue-tracker.md`.

### Triage labels

Uses the canon: `inbox`, `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context. `CONTEXT.md` at root describes the toolkit's domain model. See `docs/agents/domain.md`.

### Workflow

Technical projects and features follow the lifecycle defined in
`docs/workflow/technical-lifecycle.md`:

**Wayfinder** OR **Grill** → **to-spec** → **to-tickets** → **implement** → **code-review**

- **Wayfinder** — exploration and shaping, use when scope isn't clear
- **Grill** (`grill-with-docs` / `grill-me`) — structured interrogation, use
  when you have a direction but it needs sharpening
- **to-spec** — produce a single specification defining the destination
- **to-tickets** — decompose the spec into session-sized, dependency-tracked
  tickets
- **implement** — one ticket per session with TDD at pre-agreed seams
- **code-review** — two-axis review: coding standards and spec conformance

Do not skip phases. Each has a defined completion gate before the next begins.
