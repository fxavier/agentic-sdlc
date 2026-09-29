# Feature graph

New behaviour, from request to merge-ready branch.

## Nodes

| id | node | skill | agent | inputs | outputs | blocked-by | gate out |
|---|---|---|---|---|---|---|---|
| F0 | preflight | — | main | repo | verification commands | — | G0 |
| F1 | explore | — | Explore ×≤4 ∥ | request, `CONTEXT.md`, ADRs | affected modules, existing seams, prior-art tests | F0 | — |
| F2 | grill | grilling + domain-modeling | main | request, F1 | resolved decisions, observable acceptance criteria, `CONTEXT.md`, ADRs | F1 | G1 |
| F3 | prototype *(optional)* | prototype | main | one open question from F2 | captured answer; prototype deleted | F2 | returns to F2 |
| F4 | spec | to-prd | main | F2 | PRD issue | G1 | G2 |
| F5 | slice | to-issues | main | PRD | vertical-slice issues with `Blocked by` | G2 | G3 |
| F6.n | build slice n | tdd, or improve plan → execute | main or executor | issue n | diff in branch/worktree, green tests | G3 + slice blockers | G4 |
| F7.n | review slice n | — | fresh reviewer subagent | diff, issue n | verdict | F6.n | G5 |
| F8 | integrate | — | main | approved slices | feature branch, full suite green | all F7 | G4 |
| F9 | architecture review *(conditional)* | — | fresh reviewer subagent | full diff | verdict | F8 | G6 |
| F10 | ship | — | human | run summary | merge | F8 (F9 if run) | G7 |

## Parallelism

- F1 fans out by area (e.g. domain/persistence, API/integration, UI, tests/CI).
- F6.n run in parallel only for slices with no `Blocked by` path and disjoint write sets. Prefactor slices always go first and alone.
- Pipeline F7.n: review each slice as soon as its F6.n passes G4 — don't wait for every build.

## Build path for F6.n

- **main + tdd** when interface decisions remain — tdd's planning step asks the user to confirm the interface and the behaviours to test.
- **advisor → executor** when the slice is mechanical: `improve plan <issue>` → `plans/NNN-*.md` → `improve review-plan` (fresh context) → `improve execute` (executor in worktree, following tdd) → advisor verdict. The plan is the product; a cheaper model executes.

Record the choice and the reason per slice in the ledger.

## Size profiles

- **S** — F0 → F2 (lite: stop once acceptance criteria are observable) → F6 (main + tdd) → F7 → F10. No PRD or issues; acceptance criteria live in the ledger.
- **M** — every node except F3 unless an open question needs it; F9 only when a G6 trigger fires.
- **L** — every node. F2 must update `CONTEXT.md`. If F5 yields more than ~10 slices, split into several PRDs and several runs.