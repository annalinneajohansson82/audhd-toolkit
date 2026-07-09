# Technical Lifecycle

Canonical workflow for technical projects and features. Every phase has a
defined completion gate. Do not skip phases.

## Overview

```
[Wayfinder]  ──┐
               ├──→ to-spec → to-tickets → implement → code-review
[Grill] ───────┘
```

**Two entry points.** Wayfinder when you're exploring and the scope isn't clear.
Grill (`grill-with-docs` or `grill-me`) when you have a direction but need it
sharpened into a spec. Both feed the same pipeline.

---

## Phase 0a: Wayfinder

Use when: you don't know what you're building yet. The shape, scope, or even
the problem itself is still foggy.

Outcome: a map — what's known, what's unknown, what to investigate next. The
map lives as a GitHub issue labelled `wayfinder:map` in the project's repo.

Gate: the map has enough resolution that the next step (Grill) isn't
speculative. You can describe roughly what you're trying to do.

---

## Phase 0b: Grill (`grill-with-docs` or `grill-me`)

Use when: you have a direction but the plan is half-baked. Gaps need
exposing, trade-offs need naming, non-obvious constraints need capturing.

Use `grill-with-docs` when the project already has documentation that the
agent should read before grilling you (existing specs, ADRs, domain models).

Use `grill-me` when there's nothing to read — the agent explores the codebase
or project context, then grills you from scratch.

Outcome: a shared understanding. Key decisions captured as ADRs. Glossary
terms defined. The agent does NOT proceed until you confirm with the
confirmation gate ("don't enact the plan until I confirm we've reached a
shared understanding").

Gate: you've confirmed shared understanding. The plan is clear enough that a
spec would be straightforward to write.

---

## Phase 1: to-spec

**Skill: `to-spec`**

Takes the output of the Grill (or the current conversation) and produces a
single specification document. A spec defines the destination — what we're
building, why, and the constraints — but not the detailed implementation
steps.

Outcomes:

- A spec published as a GitHub issue on the project repo, labelled `spec`
- The spec captures: context, goals, non-goals, design decisions, open
  questions, success criteria

Gate: the spec is complete enough that tickets can be extracted from it. No
open questions that block decomposition.

---

## Phase 2: to-tickets

**Skill: `to-tickets`**

Takes the spec and decomposes it into individual tickets. Each ticket is
session-sized — something that can be implemented in a single agent session.
Tickets declare their blocking edges so work can be independent.

Outcomes:

- Tickets created as GitHub issues in the project repo
- Blocking relationships tracked via GitHub issue dependencies
- Each ticket links back to the parent spec issue

Gate: every ticket is either unblocked (can be started immediately) or
declares what it's blocked by. The set of tickets covers the spec with no
gaps that would stall implementation.

---

## Phase 3: implement

**Skill: `implement`**

One ticket per session. Implement the work described in the ticket or spec.

Guidelines:

- Use TDD where possible at pre-agreed seams
- Run type checking regularly
- Run single test files regularly
- Run the full test suite once at the end
- Commit work to the current branch

Gate: all tests pass. Code is pushed to a branch. The ticket is ready for
review.

---

## Phase 4: code-review

**Skill: `code-review`**

Reviews on two axes running as parallel sub-agents:

1. **Standards axis** — does the code conform to the project's documented
   coding standards? Reads from `CODING_STANDARDS.md` if it exists.
2. **Spec axis** — does the code faithfully implement the originating spec
   or ticket?

Uses Martin Fowler's refactoring smells as shared vocabulary: mysterious
name, duplicated code, feature envy, data clumps, primitive obsession,
repeated switches, divergent change, speculative generality, message chains.

Outcome: review notes. If issues are found, they're fixed in a follow-up
commit on the same branch. If the review passes, the branch is ready to merge.

Gate: code review passes on both axes. Branch is ready to merge.

---

## After the lifecycle

Merge the branch. Close the ticket. If this completes the spec, close the
spec issue too. Next ticket starts at Phase 3 (implement).

If new information during implementation changes the spec, go back to
Phase 0b (Grill) or Phase 1 (to-spec) depending on how fundamental the
change is.
