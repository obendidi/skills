---
name: lab
description: Tier 3 empirical research, hard-bug diagnosis, and autonomous optimization loop. Use when the user says "tier 3", "lab", "deep lab", "autoresearch", "research loop", "overnight lab", or "spike", or when investigating flaky tests, regressions, or hard bugs whose root cause stays unclear after a quick check.
---

# Lab (Tier 3)

Falsification-driven loop for uncharted problems, hard bugs, and performance or accuracy optimization. Runs beat opinions: never theorize or debate what a fast experiment can measure.

## Modes

- **Standard (`lab`, `spike`, hard bugs — default)**: 2–3 rounds for hard bugs, flakes, and localized regressions. Run Steps 1–6 below directly.
- **Deep / Autoresearch (`deep lab`, `autoresearch`, `research loop`, `overnight lab`, or open-ended multi-round metric goals)**: Load [`references/deep-loop.md`](references/deep-loop.md) before Step 1. Adds a worktree-safe `$LAB_DIR` state folder, a 3-angle literature & codebase survey before coding, multi-round frontier evolution on a `lab/<slug>` branch, an automated stall-breaker, and non-stop execution rules.

## 1. Question card & budget

If an `ambition` Reframed Brief or confirmed decisions from a finished `grill-me` session exist in this thread, inherit their goal, constraints, and out-of-scope guardrails — map rawest inputs and instruments onto Gate 0, and map any outside judge onto Signal (or Step 6 holdout if it is a holdout). Before fanning out or spending compute, state in ≤8 lines:
- **Decision**: what choice or fix this informs.
- **Signal**: the exact metric or red/green symptom and acceptance threshold.
- **Control**: base commit SHA and baseline measurement.
- **Write scope**: choose one before Gate 0:
  - `read-only` (required in Plan mode or explicit read-only asks): stop after read-only recon/profiling and propose the harness + experiment plan for approval — do not create branches, commit fixtures, or edit files.
  - `benchmark-only`: run Gate 0 + isolated experiments, then stop with verified findings and clean up scratch worktrees (skip Step 6 promotion).
  - `experiment + promote`: run full loop through Step 6 clean promotion.
- **Budget**: defaults are max **3 concurrent delegates**, max **2–3 rounds** (**8 rounds** in Deep mode), and **zero paid compute** (get explicit approval before paid GPU/cloud spend).

## 2. Gate 0 — Harness & noise floor first

Build or name **one fast, runnable command** (failing test, replay script, curl, benchmark, profiler, or telemetry query) and run it on the control before generating hypotheses:
- **Red-capable & calibrated**: fails on the exact symptom or prints `METRIC=<value>` deterministically. For flakes, raise repro rate with stress/loops first; for noisy timing/ML benchmarks, run 2–3× on control to record the noise floor (`±δ`) and require candidates to beat control beyond `±δ`.
- **Rawest source & real instruments**: run on raw logs, traces, or representative dev inputs rather than pre-digested summaries (never the promotion holdout), and use real field instruments (profiler, solver, statistical test, simulator) instead of toy proxies.
- **Minimised, shared & locked**: strip unrelated inputs/steps until every remaining piece is load-bearing. Commit any new test/fixture on a working branch as the control SHA (or put external replay scripts in shared scratch storage by absolute path) so clean worktrees have it; workers may never edit tests, fixtures, tolerances, or the eval harness.
- **Redacted**: scrub secrets (`<REDACTED>`) before showing commands or outputs.
- **No harness, no theories**: if you cannot build a runnable signal, stop, list what you tried, and ask the user for traces, fixtures, or access.

## 3. Diverge in isolated worktrees

Generate **3–5 falsifiable hypotheses** (`If X, changing Y produces Z`) rooted in the underlying mechanism and hard invariants — include at least one structural change alongside incremental tweaks. Show the ranked list to the user (don't block if they are AFK or in Deep mode).

Isolate each experiment from the committed control (or current frontier commit in Deep mode):
- **Multi-provider / worktree harnesses** (e.g. T3 Code `orchestrator_capabilities` + `delegate_task` with `isolation: "worktree"`, selecting seats per `council` § Seats — fallback if absent: 1 top-generation flagship per distinct provider at `high` effort; or Claude Code `Agent` with `isolation: "worktree"`): assign each delegate a **distinct hypothesis or failure mode** (never 3 copies of the same generic prompt).
- **Single-provider fallback**: state single-provider mode explicitly and test distinct hypotheses via fresh subagent worktrees or sequential `git worktree` checkouts from the same base SHA.
- Pass the exact eval command and require a ≤15-line reply: hypothesis, commit/diff summary (`+added/-removed`), command run, raw output/numbers, and lesson learned.

## 4. Verify & cross-examine

- **Never trust self-reported numbers**: the coordinator re-runs viable candidates' exact command in ranked order against the control baseline and verifies tests/fixtures/harness were untouched.
- **Simplicity tie-breaker**: discard complex special-case diffs that only move the metric within `±δ`; prefer code-deleting simplifications at equal metric.
- **Adversarial cross-check**: run `review` (or a fresh read-only subagent / `council` seat from a different provider than the winner) checking for benchmark-overfitted hotfixes, reward hacking, and broken callers.
- **Kill losers fast**: cancel or discard branches that miss the threshold or stall (e.g. `task_cancel` in T3 Code).

## 5. Experiment ledger & stop rules

Track every run in one compact table:

```markdown
| # | Hypothesis | Change (+/−) | Metric vs control | Verdict |
```

- **Change one variable per run**; tag temporary debug logs `[DEBUG-<id>]` so one grep cleans them all.
- **Pivot cap**: in Standard mode, if 2 consecutive runs fail to move the metric beyond `±δ`, stop and re-frame the bottleneck and baseline assumptions with the user (in Deep mode, run the automated stall-breaker in [`references/deep-loop.md`](references/deep-loop.md) first).
- **Hard stop**: stop once a verified result clears the threshold or the round cap hits. *"Inconclusive"* with verified numbers is a valid outcome.

## 6. Clean promotion

For benchmark/read-only runs, stop after reporting verified numbers and clean up scratch worktrees. When promotion is in scope, a winning spike only proves the approach:
1. Re-implement the winner minimally on a clean branch with a regression test at the real call-site seam (if no seam exists, report that architectural gap).
2. Re-run the original un-minimised Gate 0 suite or promotion holdout check to prove generalization.
3. Delete throwaway worktrees, scratch scripts/state, and `[DEBUG-...]` logs.
4. State the proven cause and measured result in the commit or PR description; ship through `pr` only when asked.
