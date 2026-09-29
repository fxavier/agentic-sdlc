# Bug graph

Broken, wrong or slow behaviour, from report to regression-locked fix. The core is diagnosing-bugs: **no red-capable command, no hypothesis.**

## Nodes

| id | node | skill | agent | inputs | outputs | blocked-by | gate out |
|---|---|---|---|---|---|---|---|
| B0 | preflight | — | main | repo | verification commands | — | G0 |
| B1 | triage *(if from tracker)* | triage | main | issue | verified claim, category/state | B0 | — |
| B2 | loop | diagnosing-bugs Phase 1–2 | main | report, B1 | red-capable command (run, output pasted), minimised repro | B0/B1 | L1 |
| B3 | hypothesise | diagnosing-bugs Phase 3 | main | B2 | 3–5 ranked falsifiable hypotheses, shown to user | L1 | — |
| B4.k | probe hypothesis k *(optional ∥)* | diagnosing-bugs Phase 4 | Explore or executor in worktree | hypothesis k, B2 command | confirmed / falsified + evidence | B3 | — |
| B5 | fix | diagnosing-bugs Phase 5 + tdd | main | confirmed hypothesis, minimised repro | regression test (red → green), fix | B4 | G4 |
| B6 | cleanup | diagnosing-bugs Phase 6 | main | B5 | no `[DEBUG-…]` tags, prototypes removed, cause in commit message | B5 | G4 |
| B7 | review | — | fresh reviewer subagent | diff, report | verdict | B6 | G5 |
| B8 | post-mortem | diagnosing-bugs Phase 6 | main | B5 findings | "what would have prevented this?"; follow-up architecture run if needed | B7 | — |
| B9 | ship | — | human | summary | merge | B7 | G7 |

**L1 (loop gate)** — pass only with one command that was run at least once and is red-capable, deterministic (or at a pinned high repro rate), fast and agent-runnable. Fail route: keep building the loop; if impossible, halt and ask for environment access, a captured artifact or permission for temporary instrumentation.

## Parallelism

- B4.k parallel only when each probe changes one variable in its own worktree and none write to shared state (DB, caches, queues). Otherwise probe sequentially in ranked order.
- Stop fanning out the moment a hypothesis is confirmed.

## Perf branch

Replace B2–B5 with:

1. **Baseline** — a benchmark harness at the right seam: JMH (Java), `go test -bench -benchmem` + `benchstat`, `pytest-benchmark`, k6/Gatling for HTTP, `EXPLAIN (ANALYZE, BUFFERS)` for PostgreSQL. Fix warm-up, input size and environment. Record the baseline over ≥5 runs with variance.
2. **Threshold** — agreed with the user (e.g. p95 < X ms, allocations < Y) before any change.
3. **Bisect / profile** — measure first; logs are usually the wrong tool (async-profiler/JFR, `pprof`, `py-spy`).
4. **Fix** — one change at a time, re-measured against the same harness.

G4 for perf: threshold met across ≥5 runs, improvement beyond run-to-run variance, full suite green.

## Post-mortem routing

If B5 found **no correct seam** for the regression test, or the answer to "what would have prevented this?" is architectural (tangled callers, hidden coupling), open a follow-up run on the architecture graph (targeted mode) with the specifics. Do it after the fix is shipped, not instead of it.

## Size profiles

- **S** — B0 → B2 → B3 → B5 → B6 → B7 → B9. No parallel probes.
- **M/L** — every node; B1 when the bug came from the tracker; B4 fan-out when ≥3 hypotheses remain plausible after ranking.