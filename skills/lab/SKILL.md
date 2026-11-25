---
name: lab
description: Tier 3 empirical research, hard-bug diagnosis, and autonomous optimization loop. Use when the user says "tier 3", "lab", "deep lab", "autoresearch", "research loop", "overnight lab", "debug", "diagnose", or "spike", or when investigating benchmark, performance, or accuracy regressions, flaky tests, or open-ended research goals that need a runnable harness and isolated experiments.
---

# Lab (Tier 3)

Falsification-driven loop for uncharted problems, hard bugs, and performance or accuracy optimization. Runs beat opinions: never theorize or debate what a fast experiment can measure.

## Modes

- **Standard (`lab`, `debug`, `diagnose`, `spike` — default)**: 2–3 rounds for hard bugs, flakes, and localized regressions. Run Steps 1–6 below directly.
- **Deep / Autoresearch (`deep lab`, `autoresearch`, `research loop`, `overnight lab`, or open-ended multi-round metric goals)**: Load [`references/deep-loop.md`](references/deep-loop.md) before Step 1. Adds a worktree-safe `$LAB_DIR` state folder, a 3-angle literature & codebase survey before coding, multi-round frontier evolution on a `lab/<slug>` branch, an automated stall-breaker, and non-stop execution rules.

## 1. Question card & budget

Before fanning out or spending compute, state in ≤8 lines:
- **Decision**: what choice or fix this informs.
- **Signal**: the exact metric or red/green symptom and acceptance threshold.
- **Control**: base commit SHA and baseline measurement.
- **Budget**: defaults are max **3 concurrent delegates**, max **2–3 rounds** (**8 rounds** in Deep mode), and **zero paid compute** (get explicit approval before paid GPU/cloud spend).

## 2. Gate 0 — Harness & noise floor first

Build or name **one fast, runnable command** (failing test, replay script, curl, benchmark, profiler, or telemetry query) and run it on the control before generating hypotheses:
- **Red-capable & calibrated**: fails on the exact symptom or prints `METRIC=<value>` deterministically. For flakes, raise repro rate with stress/loops first; for noisy timing/ML benchmarks, run 2–3× on control to record the noise floor (`±δ`) and require candidates to beat control beyond `±δ`.
- **Minimised & locked**: strip unrelated inputs/steps until every remaining piece is load-bearing. Workers may never edit tests, fixtures, tolerances, or the eval harness itself.
- **Redacted**: scrub secrets (`<REDACTED>`) before showing commands or outputs.
- **No harness, no theories**: if you cannot build a runnable signal, stop, list what you tried, and ask the user for traces, fixtures, or access.

## 3. Diverge in isolated worktrees

Generate **3–5 falsifiable hypotheses** (`If X, changing Y produces Z`). Show the ranked list to the user (don't block if they are AFK or in Deep mode).

Isolate each experiment from the committed control (or current frontier commit in Deep mode):
- **Multi-provider / worktree harnesses** (e.g. T3 Code `orchestrator_capabilities` + `delegate_task` with `isolation: "worktree"`, picking 1 latest flagship per available provider at high/max/ultra reasoning effort; or Claude Code `Agent` with `isolation: "worktree"`): assign each delegate a **distinct hypothesis or failure mode** (never 3 copies of the same generic prompt).
- **Single-provider fallback**: state single-provider mode explicitly and test distinct hypotheses via fresh subagent worktrees or sequential `git worktree` checkouts from the same base SHA.
- Pass the exact eval command and require a ≤15-line reply: hypothesis, commit/diff summary (`+added/-removed`), command run, raw output/numbers, and lesson learned.

## 4. Verify & cross-examine

- **Never trust self-reported numbers**: the coordinator re-runs the winning candidate's exact command against the control baseline and verifies tests/fixtures/harness were untouched.
- **Simplicity tie-breaker**: discard complex special-case diffs that only move the metric within `±δ`; prefer code-deleting simplifications at equal metric.
- **Adversarial cross-check**: run [`review`](../review/SKILL.md) (or a fresh read-only subagent / [`council`](../council/SKILL.md) seat from a different provider than the winner) checking for benchmark-overfitted hotfixes, reward hacking, and broken callers.
- **Kill losers fast**: cancel or discard branches that miss the threshold or stall (e.g. `task_cancel` in T3 Code).

## 5. Experiment ledger & stop rules

Track every run in one compact table:

```markdown
| # | Hypothesis | Change (+/−) | Metric vs control | Verdict |
```

- **Change one variable per run**; tag temporary debug logs `[DEBUG-<id>]` so one grep cleans them all.
- **Pivot cap**: in Standard mode, if 2 consecutive runs fail to move the metric beyond `±δ`, stop and re-frame with the user (in Deep mode, run the automated stall-breaker in [`references/deep-loop.md`](references/deep-loop.md) first).
- **Hard stop**: stop once a verified result clears the threshold or the round cap hits. *"Inconclusive"* with verified numbers is a valid outcome.

## 6. Clean promotion

A winning spike only proves the approach:
1. Re-implement the winner minimally on a clean branch with a regression test at the real call-site seam (if no seam exists, report that architectural gap).
2. Re-run the original un-minimised Gate 0 suite or holdout check to prove generalization.
3. Delete throwaway worktrees, scratch scripts/state, and `[DEBUG-...]` logs.
4. State the proven cause and measured result in the commit or PR description.
