# Triage Labels

The skills speak in terms of canonical role names. This repo uses those names
directly as label strings — there is no translation layer. When a skill
mentions a role, apply the identically named label.

Every triaged issue carries **one category role** and **one state role**.

## Category roles

| Label | Meaning |
|-------|---------|
| `bug` | Something is broken |
| `enhancement` | New feature or improvement |

## State roles

| Label | Meaning |
|-------|---------|
| `needs-triage` | Ready to be evaluated |
| `needs-info` | Waiting on more context |
| `ready-for-agent` | Fully specified, an agent (Hermes by default) can handle autonomously |
| `ready-for-human` | Needs human involvement |
| `wontfix` | Will not be actioned |

## Workflow labels

| Label | Meaning |
|-------|---------|
| `wayfinder:map` | A Wayfinder map — what's known, unknown, and what to investigate next |
| `wayfinder:research` | Wayfinder child ticket: read docs/APIs/knowledge bases (AFK) |
| `wayfinder:prototype` | Wayfinder child ticket: build a rough artifact to react to (HITL) |
| `wayfinder:grilling` | Wayfinder child ticket: resolve a decision by conversation (HITL) |
| `wayfinder:task` | Wayfinder child ticket: manual work that unblocks a decision |
| `spec` | A specification produced by `to-spec` |

## Local roles (not managed by the skills)

| Label | Meaning |
|-------|---------|
| `inbox` | Raw, unshaped idea. A pre-triage holding pen for this brain's stray thoughts — needs human processing before it enters the triage state machine. The `triage` skill does not read or set this label. |

If a skill names a role that is not in these tables, stop and ask the human —
do not invent a new label.
