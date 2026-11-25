---
name: council
description: Independent multi-provider code review or plan/RFC critique using the review rubric, with fast-tier and single-provider split-lens fallbacks. Use when the user says "council", "council of 2", "fast council", or "cheap council".
---

# Council

Independent review by strong models from different providers, applying the [`review`](../review/SKILL.md) rubric across separate seats with a clean fallback when only one provider is available. Fix known issues and run checks first so reviewers audit the real change.

## Seats

### Multi-provider (when available)

- **Default (`council`, up to 3 seats)**: one latest flagship model per available provider (e.g. Claude, Codex, Antigravity) at high reasoning effort. Do not pad beyond 1 flagship per provider with weaker models — extra lower-tier seats add noise and verification cost. In T3 Code, resolve models via `orchestrator_capabilities` and launch read-only delegates in parallel via `delegate_task`.
- **`council of 2`**: two flagship providers, excluding the provider that authored the change.
- **`fast council` / `cheap council`**: mid-tier fast models across 2–3 providers for routine PRs when flagship latency/cost is unnecessary.
- If the user explicitly names a provider or model that is unavailable, report it instead of silently substituting.

### Single-provider fallback

When only one provider is available (and the user did not request a specific missing provider):
1. **State the fallback explicitly** before reviewing (same-family models share blind spots, so never present this as true multi-provider independence).
2. Launch **2 fresh, read-only subagents** (no author conversation history) with **split lenses** — pairing 1 flagship model with 1 faster sibling model when available:
   - **Seat A (flagship)**: deep bugs, security/data safety (input tracing), and design elegance vs. hotfix.
   - **Seat B (fast / second subagent)**: broken callers, spec & value/adoption alignment, scale/tests, and bloat/anti-overengineering.
3. If the harness has no subagent tool, run one cold-read self-review against `git diff <base>` using the `review` skill rubric.

Never edit the checkout while reviewers are reading it.

## Seat Briefs

### Code review brief (uses `review` rubric)

```text
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Apply the `review` skill rubric (read callers/tests; check correctness & spec, value & adoption, high-confidence security after tracing untrusted vs server-controlled input, design elegance vs hotfix, scale/tests, and lean/anti-bloat).
Skip style unless it hides a bug. Every finding needs a concrete case or cleaner seam.
blocker = intended case wrong, exploitable security, or data loss. should = real bug, missing spec, ugly hotfix with clear better seam, risky untested code, or bloat. nit = rest.
Reply:
SCORE: <n>/10
VERDICT: APPROVE or CHANGES
SUMMARY: <1 line: clean design vs tactical hotfix + worth it?>
C1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion>
```

### Plan / RFC / design critique brief (non-code)

```text
Critique only. Do not edit files or start subagents.
Target: <path or inline spec> Goal: <goal> Constraints: <constraints>
Challenge assumptions, problem fit (is it worth building, how is adoption measured?), simpler/more elegant alternatives, failure modes, and hidden complexity.
blocker = flawed premise, unworkable design, or severe risk. should = material gap or simpler architecture. nit = minor polish.
Reply:
SCORE: <n>/10
VERDICT: APPROVE or REVISE
C1 [blocker|should|nit] <section> <claim>. Fix: <suggestion>
```

## Acting on Findings

Verify every finding against the source yourself; re-classify rare cases or minor costs as nits.
- **Agree**: make the smallest fix (prefer deleting or simplifying code over adding guards or wrappers).
- **Disagree**: cite `file:line`, test, or output.
- **Out of scope / nit**: decline in one line and list as a follow-up.

## Rounds & Report

- Budget: **2 rounds** (3 only for auth, security, data loss, or concurrency with an open blocker/should). Re-reviews use a fresh reviewer/delegate with the ledger of `fixed:`, `disagree:`, and `declined:` items.
- Stop early when a round returns only nits. If disputed after 2 replies or budget ends, present both sides in 2–3 lines for the user to decide.
- Report a compact score table per round (`no score` if omitted; never invent one) plus fixed items, disagreements, and follow-ups. Add to a PR description only if asked.
