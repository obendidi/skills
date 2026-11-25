---
name: lab
description: Tier 3 empirical research and hard-bug loop. Use when the user says "tier 3", "lab", "debug", "diagnose", or "spike", or when investigating benchmark, performance, or accuracy regressions, flaky tests, or competing technical hypotheses that need a runnable harness and isolated experiments.
---

# Lab (Tier 3)

Falsification-driven loop for uncharted problems, hard bugs, and performance regressions. Runs beat opinions: never theorize or debate what a fast experiment can measure.

## 1. Question card & budget

Before fanning out or spending compute, state in ≤8 lines:
- **Decision**: what choice or fix this informs.
- **Signal**: the exact metric or red/green symptom and acceptance threshold.
- **Control**: base commit SHA or baseline measurement.
- **Budget**: defaults are max **3 concurrent delegates**, max **2–3 rounds**, **zero paid compute** (get explicit approval before paid GPU/cloud spend).

## 2. Gate 0 — Harness first

Build or name **one fast, runnable command** (failing test, replay script, curl, benchmark, profiler, or telemetry query) and run it on the control before generating hypotheses:
- **Red-capable**: fails on this exact symptom / prints the target metric deterministically (for flakes, raise repro rate with stress/loops first).
- **Minimised**: for bugs, strip unrelated inputs/steps until every remaining piece is load-bearing.
- **Redacted**: scrub secrets (`<REDACTED>`) before showing commands or outputs.
- **No harness, no theories**: if you cannot build a runnable signal, stop, list what you tried, and ask the user for traces, fixtures, or access.

## 3. Diverge in isolated worktrees

Generate **3–5 falsifiable hypotheses** (`If X, changing Y produces Z`). Show the ranked list to the user (don't block if they are AFK).

When exploring competing approaches or hard unknowns, isolate each experiment from the same committed control:
- **Multi-provider / worktree harnesses** (e.g. T3 Code `orchestrator_capabilities` + `delegate_task` with `isolation: "worktree"`, or Claude Code `Agent` with `isolation: "worktree"`): assign each delegate a **distinct hypothesis or failure mode** (never 3 copies of the same generic prompt).
- **Single-agent CLIs**: test hypotheses sequentially in temporary `git worktree` checkouts (or clean branches) from the same control SHA.
- Pass the exact eval command and require a strict ≤15-line reply: hypothesis, branch/diff summary, command run, raw output/numbers, and artifact paths.

## 4. Verify & cross-examine

- **Never trust self-reported numbers**: the parent re-runs the winning candidate's exact command against the control baseline.
- **Adversarial cross-check**: have a different provider or a fresh read-only subagent attack the winning result's assumptions or test a counterexample (or run `council` on the concrete diff and numbers).
- **Kill losers fast**: cancel or discard branches that miss the threshold or stall (e.g. `task_cancel` in T3 Code).

## 5. Experiment ledger & stop rules

Track every run in one compact table:

```markdown
| # | Hypothesis | Change | Metric vs control | Verdict |
```

- **Change one variable per run**; tag any temporary debug logs `[DEBUG-<id>]` so one grep cleans them all.
- **Pivot cap**: if 2 consecutive runs fail to move the metric, stop and re-frame with the user.
- **Hard stop**: stop once a reproduced result clears the threshold or the round cap hits. *"Inconclusive"* with verified numbers is a valid outcome.

## 6. Clean promotion

A winning spike only proves the approach:
1. Re-implement the winner minimally on a clean branch with a regression test at the real call-site seam (if no seam exists, report that architectural gap).
2. Re-run the original un-minimised Gate 0 loop to prove it passes.
3. Delete throwaway worktrees, scratch scripts, and `[DEBUG-...]` logs.
4. State the proven cause and measured result in the commit or PR description.
