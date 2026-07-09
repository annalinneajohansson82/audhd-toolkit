# Agent guide

Start here. This file is the entry point for any agent working in this repo,
regardless of harness or model. Everything below points to the canonical doc
for each rule — read the doc before acting on the rule; don't work from this
summary alone.

## Agent skills

This block is the contract the `setup-matt-pocock-skills` skill reads and
updates in place. Keep the three sub-sections below; add new guidance as
additional sections, not by rewriting these headings.

### Issue tracker

Issues live in GitHub Issues on this repo. External PRs are not a request
surface. See `docs/agents/issue-tracker.md`.

### Triage labels

Canonical role names are used directly as label strings — category roles
(`bug`, `enhancement`) plus state roles (`needs-triage`, `needs-info`,
`ready-for-agent`, `ready-for-human`, `wontfix`). `inbox` is a local
pre-triage role; `wayfinder:*` and `spec` come from the workflow. See
`docs/agents/triage-labels.md`.

### Domain docs

Single-context. `CONTEXT.md` at root is the canonical description of the
toolkit's domain model. See `docs/agents/domain.md`.

## Workflow

Technical work in the component repos follows the lifecycle:

**Wayfinder** OR **Grill** → **to-spec** → **to-tickets** → **implement** → **code-review**

Phases, gates, and entry criteria are defined in
`docs/workflow/technical-lifecycle.md`. Read it before starting technical
work. Do not skip phases.

## Skills

The skill names used above (`wayfinder`, `grill-me`, `grill-with-docs`,
`to-spec`, `to-tickets`, `implement`, `code-review`, `triage`,
`setup-matt-pocock-skills`) come from the
[matt-pocock-skills fork](https://github.com/annalinneajohansson82/matt-pocock-skills),
installed globally in Hermes (the primary agent for this toolkit). Agents on
other harnesses should load them from that repo. Those skills read the
`## Agent skills` config above and the `docs/agents/*` files it points to —
keep the two in sync.
