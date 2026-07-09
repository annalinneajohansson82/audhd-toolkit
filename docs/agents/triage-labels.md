# Triage Labels

This repo uses the canonical role names directly as label strings — there is
no translation layer. When a skill mentions a role, apply the identically
named label.

| Label | Meaning |
|-------|---------|
| `inbox` | Raw, unshaped idea. Needs human processing before it enters the pipeline |
| `needs-triage` | Ready to be evaluated |
| `needs-info` | Waiting on more context |
| `ready-for-agent` | Fully specified, an agent (Hermes by default) can handle autonomously |
| `ready-for-human` | Needs human involvement |
| `wontfix` | Will not be actioned |
| `wayfinder:map` | A Wayfinder map — what's known, unknown, and what to investigate next |
| `spec` | A specification produced by `to-spec` |

If a skill names a role that is not in this table, stop and ask the human —
do not invent a new label.
