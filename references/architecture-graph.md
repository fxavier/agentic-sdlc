# Architecture graph

Structural change: deepening modules, moving seams, paying down tech debt. Behaviour must not change unless a node says so.

## Entry modes

- **targeted** — the friction is known (bug post-mortem, user complaint, a specific module). Start at A2.
- **survey** — find the friction first. Two options:
  - `improve-codebase-architecture` — deepening opportunities as a visual report, then grilling on the one picked. Use for design/testability friction.
  - `improve` (full audit, or a focus like `tests`) — prioritised findings across bugs, security, perf and debt. Use when the question is "what should we fix next?".

## Nodes

| id | node | skill | agent | inputs | outputs | blocked-by | gate out |
|---|---|---|---|---|---|---|---|
| A0 | preflight | — | main | repo | verification commands | — | G0 |
| A1 | survey *(survey mode)* | improve-codebase-architecture or improve | main + Explore ∥ | `CONTEXT.md`, ADRs | report / findings table; user picks candidate(s) | A0 | human pick |
| A2 | grill the candidate | grilling + domain-modeling | main | candidate | constraints, what sits behind the seam, what tests survive, `CONTEXT.md` terms | A1 or A0 | G1 |
| A3 | design it twice *(M/L)* | codebase-design (DESIGN-IT-TWICE) | parallel subagents, read-only | A2 | 2–3 radically different interfaces, compared on depth, locality, seam | A2 | human choice |
| A4 | decision record | domain-modeling | main | chosen interface | ADR — only if hard to reverse, surprising, and a real trade-off | A3 | — |
| A5 | characterisation tests | tdd | main | chosen seam | tests at the new seam, **green on the old code** | A4 | G4 |
| A6 | plan | improve `plan` + `review-plan` | main + fresh subagent | A2–A5 | `plans/NNN-*.md` in dependency order | A5 | G2 |
| A7.n | execute plan n | improve `execute` | executor in worktree | plan n | diff + advisor verdict | A6 + plan dependencies | G4, G5 |
| A8 | verify | — | fresh reviewer subagent | full diff | characterisation tests unchanged and green; G6 verdict | all A7 | G6 |
| A9 | ship | — | human | summary | merge | A8 | G7 |

## Rules specific to this graph

- **Characterisation first.** No structural change before A5 is green on the unchanged code. A refactor that has to edit characterisation tests changed behaviour — stop and go back to A2.
- **Replace, don't layer.** When a new seam replaces an old one, the plan deletes the old path in the same run, or records the removal as an explicit follow-up issue.
- **ADRs are settled.** A candidate that contradicts an ADR only proceeds if A2 concluded the ADR should be reopened; then A4 supersedes it.
- A7.n run sequentially by default — refactors collide on shared files. Parallel only when the parallel rules hold and the plans' `README.md` shows no dependency.

## Size profiles

- **S** — targeted, one module: A0 → A2 → A5 → A7 (or main + tdd refactor step) → A8 → A9.
- **M** — skip A3 when only one interface shape is credible; record why.
- **L** — every node; split into several runs if `plans/README.md` exceeds ~6 plans.