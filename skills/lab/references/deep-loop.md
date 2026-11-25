# Deep Lab / Autoresearch Loop

Long-horizon extension of [`lab`](../SKILL.md) for open-ended optimization, algorithm/model search, and multi-round empirical investigations (`deep lab`, `autoresearch`, `research loop`, `overnight lab`).

## 1. Coordinator Role & Non-Stop Contract

- **Coordinator-only parent**: Keep the parent thread clean across 8+ rounds by delegating heavy literature reading, profiling, and edit-test cycles to fresh subagents/delegates. Only compact summaries and ledger rows enter parent context.
- **Non-Stop Contract**: Once Gate 0 passes and paid compute remains `$0`, **do not pause between rounds to ask "should I continue?"**. Keep looping until:
  1. Coordinator-verified metric clears the target threshold beyond noise floor `±δ`,
  2. Round or wall-clock budget hits (default: **8 rounds**, max **3 delegates/round**),
  3. Two automated stall-breaker re-frames fail to advance the frontier, or
  4. Paid cloud/GPU spend or a destructive external action requires user approval.

## 2. Worktree-Safe State (`$LAB_DIR`)

Uncommitted files inside a main checkout are invisible to `isolation: "worktree"` children and dirty `git status`. Store run state inside Git's shared common directory (outside every working tree, readable from every local worktree):

```bash
LAB_DIR="$(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null || echo /tmp)/lab/<slug>"
mkdir -p "$LAB_DIR"
```

**Coordinator-only writes** (read-only sandboxes cannot write files, and parallel workers race on appends — delegates return text; only the coordinator writes `$LAB_DIR`):
- `brief.md`: decision, target metric & direction, control SHA, baseline `±δ`, `EVAL_SHA`, guardrails (runtime/memory/test suite), round cap.
- `eval.sh`: executable script (`chmod +x`) that runs against `$PWD` (never `$(dirname "$0")`), prints `METRIC=<number>` plus guardrail status, and exits non-zero on broken tests/invariants. Record `EVAL_SHA="$(shasum -a 256 "$LAB_DIR/eval.sh" | awk '{print $1}')"` before Round 1.
- `literature.md`: synthesized prior art, citations, codebase bottlenecks, and dead ends.
- `ledger.tsv`: append-only log (`round`, `seat`, `hypothesis`, `commit`, `diff_stat`, `metric`, `delta_vs_best`, `verdict`, `lesson`).

## 3. Seats (Multi-Provider & Single-Provider Fallback)

- **Multi-provider (T3 Code)**: Call `orchestrator_capabilities` once. Pick **1 latest flagship per available provider** (do not pad with weaker models) and enable each provider's deep-reasoning / autonomous options in `target.options` when offered (e.g. `ultracode: true` + `effort: "max"`/`"xhigh"` on Claude, `reasoningEffort: "xhigh"`/`"ultra"` on Codex, `thought_level: "xhigh"` on Antigravity).
- **Single-provider fallback (Claude Code, Codex CLI, Antigravity, OpenCode, Cursor)**: State single-provider mode explicitly. Use fresh subagents (no shared conversation bias) with **disjoint research lenses** in Phase 0 and **isolated worktrees** (`isolation: "worktree"` or `git worktree add`) in Phase 1..N.
- **Network routing**: Assign web/paper searches only to delegates with network access (e.g. Antigravity's built-in `research` subagent is offline — route external angles to cross-provider or network-enabled subagents, and give offline subagents local codebase/profiler angles).

## 4. Phase 0 — Literature, Prior Art & Bottleneck Survey (Read-Only)

Before editing code, launch up to **3 parallel read-only delegates** on disjoint angles:
1. **Angle A — Literature & algorithmic SOTA**: papers, theoretical bounds, known domain failure modes, tricks that generalize vs. overfit.
2. **Angle B — OSS & ecosystem prior art**: how mature libraries, benchmarks, or upstream issues/PRs handle this exact problem.
3. **Angle C — Codebase & profiler anatomy**: profile Gate 0, map hot paths, data layout, callers, and hard invariants.

```text
Research only. Do not edit files, commit, or start subagents.
Repo: <abs path> | Angle: <A, B, or C> | Brief: <LAB_DIR>/brief.md
Find concrete, falsifiable mechanisms that move METRIC without breaking guardrails. Cite URLs (external) or file:line + profiler numbers (local). Flag dead ends.
Reply in <=25 lines:
FINDINGS: 3-5 bullets with citations/measurements
HYPOTHESES: 2-3 ranked `If X, changing Y produces Z` proposals with expected gain and risk
DEAD_ENDS: approaches to skip and why
```

Synthesize replies into `$LAB_DIR/literature.md` and rank 5–8 falsifiable hypotheses before Round 1.

## 5. Phase 1..N — Evolutionary Worktree Loop (`lab/<slug>`)

Create working branch `lab/<slug>` from the control SHA (leaving the user's branch untouched until Clean Promotion).
- **Frontier rule**: `isolation: "worktree"` branches from the coordinator's **current committed `HEAD`**. Whenever a verified candidate wins, commit/cherry-pick it onto `lab/<slug>` *before* launching Round `k+1` so new worktrees evolve from the new best state.

Each round, launch **2–3 parallel worktree delegates** on **distinct hypotheses**:

```text
Isolated experiment in your own worktree. Do not push, switch branches, or start subagents.
Read first: <LAB_DIR>/brief.md, <LAB_DIR>/literature.md, <LAB_DIR>/ledger.tsv
Assigned hypothesis: <If X, changing Y produces Z>
Do NOT repeat DISCARDed ideas in ledger.tsv or modify tests, fixtures, tolerances, or <LAB_DIR>/eval.sh.
Run `<LAB_DIR>/eval.sh` from your worktree root ($PWD). Iterate locally up to 4 edit->eval cycles to debug wiring or tune one variable cleanly. Strip any `[DEBUG-...]` logs and `git commit` your best clean state locally.
Reply in <=15 lines:
HYPOTHESIS: <1 line>
COMMIT: <sha or NONE> | DIFF_STAT: <+added/-removed>
RAW_EVAL: <exact METRIC and guardrail output>
LESSON: <what caused the delta or why it failed>
```

**Coordinator verification & selection each round**:
1. Confirm `shasum -a 256 "$LAB_DIR/eval.sh"` still equals `EVAL_SHA` and inspect `git show --stat <commit>` (auto-`DISCARD` any test, fixture, tolerance, or harness edit).
2. Apply/checkout the top candidate on `lab/<slug>` and **re-run `"$LAB_DIR/eval.sh"` yourself**.
3. **Selection (`KEEP` vs `DISCARD`)**: `KEEP` on `lab/<slug>` only if all guardrails pass and the metric beats the current frontier **beyond `±δ`** — or matches within `±δ` while **deleting net code**. Otherwise `git reset --hard` `lab/<slug>` to the prior frontier commit (`DISCARD`).
4. Append all candidates (with `LESSON`) to `$LAB_DIR/ledger.tsv` and cancel stalled tasks (`task_cancel` in T3 Code).

## 6. Automated Stall-Breaker (After 2 Flat Rounds)

If 2 consecutive rounds produce zero `KEEP` commits, launch **1 fresh read-only diagnostic delegate** (prefer a provider that did not author recent attempts):

```text
Diagnose & re-frame only. Do not edit files or start subagents.
Read: <LAB_DIR>/brief.md, <LAB_DIR>/literature.md, <LAB_DIR>/ledger.tsv, and `git log -p <control_sha>..HEAD` in <repo>.
Identify why the last two rounds plateaued (wrong bottleneck, ceiling hit, interaction bug, or variance) and propose 3 orthogonal hypotheses — including combining independent partial wins from earlier rounds.
Reply in <=20 lines: BOTTLENECK, 3 NEW HYPOTHESES, or CEILING_REACHED (with evidence).
```

Update `literature.md` and resume. If a second stall-breaker still produces 2 flat rounds, stop and report verified findings.

## 7. Anti-Overfit Audit & Clean Promotion

Before promoting `lab/<slug>` to a clean branch (Step 6 of [`SKILL.md`](../SKILL.md)):
1. Re-run the **full un-minimised test/benchmark/holdout suite** against the same-day control baseline.
2. Run [`review`](../../review/SKILL.md) — or a 2-seat [`council`](../../council/SKILL.md) excluding the winning provider — targeting **benchmark-overfitted hotfixes, reward hacking, and bloat**.
3. Re-implement the minimal winning diff on the target branch with a regression test, delete `lab/<slug>`, throwaway worktrees, and `$LAB_DIR`, and report the `ledger.tsv` summary.

## 8. Host Tips for Unattended Multi-Hour Runs

- **T3 Code**: Set high/max effort on the coordinator thread (`ultracode: true` + `max` on Claude, `xhigh`/`ultra` on Codex, `/goal` or `xhigh` on Antigravity).
- **Standalone CLIs**: Pair `"run deep lab on ..."` with your CLI's long-horizon mode (`/goal` in Antigravity/Codex, `ultracode` or `/loop` in Claude Code) for unattended overnight runs.
