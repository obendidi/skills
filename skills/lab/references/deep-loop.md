# Deep Lab / Autoresearch Loop

Long-horizon extension of [`lab`](../SKILL.md) for open-ended optimization, algorithm/model search, and multi-round empirical investigations (`deep lab`, `autoresearch`, `research loop`, `overnight lab`).

**Phase order & standalone mode**: Run `lab` Steps 1–2 on `lab/<slug>` → `deep-loop` §2–6 → `deep-loop` §7 + `lab` Step 6. On CLIs without subagents, the coordinator runs Phase 0 and isolated `git worktree` experiments serially and discloses the loss of independent cross-provider review.

## 1. Coordinator Role & Non-Stop Contract

- **Coordinator-only parent**: Keep the parent thread clean across 8+ rounds by delegating heavy literature reading, profiling, and edit-test cycles to fresh subagents/delegates. Only compact summaries and ledger rows enter parent context.
- **Non-Stop Contract**: Once Gate 0 passes, write scope authorizes experiments (not Plan mode / read-only), and paid compute remains `$0`, **do not pause between rounds to ask "should I continue?"**. Keep looping until:
  1. **Goal or ceiling met**: coordinator-verified metric clears an explicit target threshold beyond `±δ` — or, on open-ended goals (after ≥2 rounds), captures **≥85–90% of the removable gap** to a comparable `reference_ceiling` measured under matching workload and constraints (when no comparable ceiling exists, rely on budget and stall caps),
  2. Round or wall-clock budget hits (default: **8 rounds**, max **3 delegates/round**),
  3. Verified `CEILING_REACHED` or the stall/diminishing-returns cap hits (Section 6), or
  4. Paid cloud/GPU spend or a destructive external action requires user approval.

## 2. Worktree-Safe State (`$LAB_DIR`)

Uncommitted files inside a main checkout are invisible to `isolation: "worktree"` children and dirty `git status`. Store transient run state inside Git's shared common directory when writable, falling back to a deterministic path in `$TMPDIR` when sandboxes write-protect `.git/`:

```bash
COMMON_GIT="$(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null || git rev-parse --git-common-dir 2>/dev/null || true)"
REPO_KEY="$(printf '%s' "${COMMON_GIT:-$PWD}" | (sha256sum 2>/dev/null || shasum -a 256) | cut -c1-8)"
LAB_DIR="${COMMON_GIT:+$(cd "$COMMON_GIT" && pwd)/lab/<slug>}"
if [[ -z "$LAB_DIR" ]] || ! mkdir -p "$LAB_DIR" 2>/dev/null; then
  LAB_DIR="${TMPDIR:-/tmp}/lab-${REPO_KEY}-<slug>"
  mkdir -p "$LAB_DIR"
fi
```

**Coordinator-only writes** (read-only sandboxes cannot write files, and parallel workers race on appends — delegates return text; only the coordinator writes `$LAB_DIR`):
- `brief.md`: decision, write scope, target metric & direction (or `open` + comparable `reference_ceiling` + optional `theoretical_floor`), control SHA, baseline `±δ`, `EVAL_SHA`, guardrails (runtime/memory/test suite), round cap, and `cooled_down_providers` (`<provider>@round`).
- `eval.sh`: executable script (`chmod +x`) that runs against `$PWD` (never `$(dirname "$0")`), prints `METRIC=<number>` plus guardrail status, and exits non-zero on broken tests/invariants. Record `EVAL_SHA="$((sha256sum "$LAB_DIR/eval.sh" 2>/dev/null || shasum -a 256 "$LAB_DIR/eval.sh") | awk '{print $1}')"` and assert `[[ ${#EVAL_SHA} -eq 64 ]]` before Round 1.
- `literature.md`: synthesized prior art, citations, codebase bottlenecks, and dead ends.
- `ledger.tsv`: append-only log (`round`, `seat`, `hypothesis`, `applied_sha`, `diff_stat`, `metric`, `delta_vs_best`, `verdict`, `lesson`) where `applied_sha` is the **actual commit SHA on `lab/<slug>`** for `KEEP` rows (or candidate SHA/`NONE` for `DISCARD`).
- **Mid-run resume**: If `$LAB_DIR/brief.md` and `$LAB_DIR/ledger.tsv` already exist (e.g. after switching the parent thread's provider/model on a quota hit), compare `lab/<slug>` `HEAD` against the last `KEEP` `applied_sha` in `ledger.tsv` (if `HEAD` is ahead due to an interrupted write, verify it with `eval.sh` and append the missing `KEEP` row before continuing; otherwise check out the last `KEEP` SHA or control SHA), read `literature.md`, and resume at the next unfinished round without repeating Gate 0 or Phase 0.

## 3. Seats, Effort Pacing & Quota Failover

- **Seats & peer-flagship rotation**: Select seats per `council` § Seats (fallback if absent: 1 top-generation flagship per distinct model family at `high` effort; never pad with older or fast/mini tiers). When a provider exposes multiple **peer top-tier flagships**, **rotate peer variants across rounds** so both get exercised, and use a second peer flagship to fill quota-vacated seats.
- **Progressive effort escalation (pace quota, save big guns for stalls)**:
  - **Cruise (Phase 0 & normal rounds)**: Run child delegates at **`high` effort with nested orchestration off** (e.g. Claude `effort: "high"`, `ultracode: false`; Codex `reasoningEffort: "high"`; Antigravity `thought_level: "high"`).
  - **Big guns (Stalls only)**: Escalate child delegates to **`xhigh` effort** (still keeping `ultracode: false` on all child seats so they never spawn nested subagents) **only** on the automated stall-breaker (Section 6) and the round immediately following a plateau. Reserve `max` / `ultra` / `ultrathink` for explicit user requests.
- **Quota / rate-limit failover (`3 → 2 → 1`)**: Unless the user pinned a specific provider/model, on quota/`429` failure record `<provider>@<round>` in `cooled_down_providers` (retry after 2 rounds for transient rate limits; skip permanently on hard cap exhaustion), log a 1-line note in `ledger.tsv`, and re-dispatch the hypothesis to a healthy provider or peer flagship (`3 → 2 → 1`; pause only if all providers are exhausted).
- **Network routing**: Assign web/paper searches only to delegates with network access (e.g. Antigravity's built-in `research` subagent is offline — route external angles to cross-provider or network-enabled subagents, and give offline subagents local codebase/profiler angles).

## 4. Phase 0 — Literature, Prior Art & Bottleneck Survey (Read-Only)

Before editing code, launch up to **3 parallel read-only delegates** on disjoint angles:
1. **Angle A — Literature, SOTA & adjacent fields**: papers, theoretical bounds, known failure modes, and — when standard domain tricks look tapped out — 1–2 unrelated fields solving the same domain-free verb (concrete methods only, never persona roleplay).
2. **Angle B — OSS & ecosystem prior art**: how mature libraries, benchmarks, or upstream issues/PRs handle this exact problem.
3. **Angle C — Codebase & profiler anatomy**: profile Gate 0, map hot paths, data layout, callers, and hard invariants; separate required invariants from accidental baseline assumptions capping the metric.

```text
Research only. Do not edit files, commit, or start subagents.
Repo: <abs path> | Angle: <A, B, or C> | Brief: <LAB_DIR>/brief.md
Find concrete, falsifiable mechanisms that move METRIC without breaking guardrails. Separate hard invariants from accidental baseline assumptions. Cite URLs (external) or file:line + profiler numbers (local). Flag dead ends.
Reply in <=28 lines:
FINDINGS: 3-5 bullets with citations/measurements
ASSUMPTIONS: <hard invariants vs. accidental baseline assumptions that may cap METRIC>
THEORETICAL_FLOOR: <physical/hardware/information-theoretic bound if measurable>
REFERENCE_CEILING: <comparable SOTA or oracle on matching workload/constraints, or unknown>
HYPOTHESES: 2-3 ranked `If X, changing Y produces Z` proposals with expected gain and risk
DEAD_ENDS: approaches to skip and why
```

Synthesize replies into `$LAB_DIR/literature.md`, record `theoretical_floor` (ambition anchor) and comparable `reference_ceiling` (stopping target, or `unknown`) in `brief.md`, and rank 5–8 falsifiable hypotheses mixing structural bets with incremental wins.

## 5. Phase 1..N — Evolutionary Worktree Loop (`lab/<slug>`)

Require `git status --porcelain` to be empty before creating/switching to working branch `lab/<slug>` from the control SHA (or run the coordinator in its own dedicated worktree so user WIP is never touched).
- **Frontier rule**: `isolation: "worktree"` branches from the coordinator's **current committed `HEAD`**. Whenever a verified candidate wins, advance `lab/<slug>` *before* launching Round `k+1` so new worktrees evolve from the new best state.

Each round, launch **2–3 parallel worktree delegates** on **distinct hypotheses** (avoid spending a whole round on knob tweaks while structural baseline assumptions remain untested):

```text
Isolated experiment in your own worktree. Do not push, switch branches, or start subagents.
Read first: <LAB_DIR>/brief.md, <LAB_DIR>/literature.md, <LAB_DIR>/ledger.tsv
Assigned hypothesis: <If X, changing Y produces Z>
Reason from the underlying mechanism and invariants — do not hardcode eval quirks or bolt on symptom workarounds. Do NOT repeat DISCARDed ideas in ledger.tsv or modify tests, fixtures, tolerances, or <LAB_DIR>/eval.sh.
Run `<LAB_DIR>/eval.sh` from your worktree root ($PWD). Iterate locally up to 4 edit->eval cycles to debug wiring or tune one variable cleanly. Strip any `[DEBUG-...]` logs and `git commit` your best clean state locally (if sandbox rules block `git commit`, leave clean edits in place and report `WORKTREE: <abs path>`).
Reply in <=15 lines:
HYPOTHESIS: <1 line>
COMMIT: <sha or WORKTREE:<abs path> or NONE> | DIFF_STAT: <+added/-removed>
RAW_EVAL: <exact METRIC and guardrail output>
LESSON: <what caused the delta or why it failed>
```

**Coordinator verification & selection each round**:
1. Re-compute `CURRENT_SHA="$((sha256sum "$LAB_DIR/eval.sh" 2>/dev/null || shasum -a 256 "$LAB_DIR/eval.sh") | awk '{print $1}')"` and assert `[[ ${#CURRENT_SHA} -eq 64 && "$CURRENT_SHA" == "$EVAL_SHA" ]]`.
2. Test viable candidates **in ranked order** inside a disposable detached worktree (`VERIFY_WT="$(mktemp -d)" && git worktree add --detach "$VERIFY_WT" <commit>`, or `git worktree add --detach "$VERIFY_WT" <frontier_sha>` + apply `git -C <worktree_path> diff` for uncommitted `WORKTREE:<path>` winners): inspect `git -C "$VERIFY_WT" diff --stat <frontier_sha>` (auto-`DISCARD` any test, fixture, tolerance, or harness edit) and **re-run `"$LAB_DIR/eval.sh"` from `"$VERIFY_WT"` yourself** until one qualifies or all fail; remove `"$VERIFY_WT"` (`git worktree remove --force "$VERIFY_WT"`).
3. **Selection (`KEEP` vs `DISCARD`)**: `KEEP` only if all guardrails pass and the metric beats the current frontier **beyond `±δ`** — or matches within `±δ` while **deleting net code**. Advance `lab/<slug>` **only after** a candidate qualifies (`git merge --ff-only <commit>`, `git cherry-pick <commit>`, or apply + `git commit` for an uncommitted `WORKTREE:<path>` winner; never `git reset --hard` on a shared checkout) and capture `APPLIED_SHA="$(git rev-parse HEAD)"`.
4. Immediately append all candidates (recording `APPLIED_SHA` for `KEEP` rows and `LESSON` for every row) to `$LAB_DIR/ledger.tsv` before starting the next round, and cancel stalled tasks (`task_cancel` in T3 Code).

## 6. Diminishing Returns, Stall-Breaker & Open-Ended Early Exit

- **Minimum 2 rounds on open-ended goals**: Never exit after Round 1 without an explicit user threshold — Round 1 only harvests single-variable wins before testing whether orthogonal wins combine.
- **Crumbs count as flat**: Once a meaningful `KEEP` win is banked and ≥2 rounds have run, treat any round whose gain is **`< 10%` of cumulative gain so far** as a **flat round** so the loop stops grinding marginal crumbs.

After **2 consecutive flat/crumb rounds**, launch **1 fresh read-only diagnostic delegate** at `xhigh` effort (prefer a provider that did not author recent attempts) for a **mechanism reset**:

```text
Diagnose & re-frame only. Do not edit files or start subagents.
Read: <LAB_DIR>/brief.md, <LAB_DIR>/literature.md, <LAB_DIR>/ledger.tsv, and `git log -p <control_sha>..HEAD` in <repo>.
Strip the problem back to its raw goal, hard invariants, and true bottleneck. Name which hidden baseline assumption is capping METRIC, and propose 3 orthogonal hypotheses (challenging untested assumptions, combining independent partial wins, and — when the bottleneck isn't a local profiler hotspot — importing 1 concrete method from an adjacent field via Phase 0's verb translation, never persona roleplay).
Reply in <=22 lines: BOTTLENECK, QUESTIONED_ASSUMPTIONS, 3 NEW HYPOTHESES, or CEILING_REACHED (with concrete profiler/bound evidence).
```

- **Immediate ceiling exit**: If the delegate returns `CEILING_REACHED` backed by concrete profiler, physical-bound, or data-ceiling evidence verified by the coordinator (never just "known tweaks stalled"), **stop immediately** and proceed to Section 7.
- **Asymmetric stall cap**: If `lab/<slug>` **already banked a verified win** over control, run **at most 1 stall-breaker + 1 post-stall round** (exit if still flat/crumb). Allow a **2nd stall-breaker** only when still stuck at baseline (`0` wins); if 2 post-stall rounds remain flat, stop and report verified findings.

## 7. Anti-Overfit Audit & Clean Promotion

Before promoting `lab/<slug>` (Step 6 of [`SKILL.md`](../SKILL.md)):
1. Re-run the **full un-minimised test/benchmark/holdout suite** against the same-day control baseline.
2. Run `review` — or a 2-seat `council` excluding the winning provider — targeting **benchmark-overfitted hotfixes, reward hacking, and bloat**.
3. If the repo's main checkout has `output/research/README.md`, copy durable write-ups and `ledger.tsv` to `output/research/<topic>/` before deleting `$LAB_DIR`.
4. Report the `ledger.tsv` summary and verified control-vs-winner metrics first. If promotion is in scope, re-implement the minimal winning diff on the target branch with a regression test and clean up throwaway worktrees (`lab/<slug>` and `$LAB_DIR` once confirmed).

## 8. Host Tips for Unattended Multi-Hour Runs

- **T3 Code**: Pace the coordinator thread at `high` effort (e.g. `effort: "high"` or `medium` + `ultracode: true` on Claude, `high` on Codex, `/goal` or `high` on Antigravity) — avoid `max`/`ultra` on multi-round coordinators so usage limits last the full run. If the coordinator hits a quota cap mid-run, switch the thread's provider/model and say `"resume deep lab"`.
- **Standalone CLIs**: Pair `"run deep lab on ..."` with your CLI's long-horizon mode (`/goal` in Antigravity/Codex, `ultracode` or `/loop` in Claude Code) for unattended overnight runs.
