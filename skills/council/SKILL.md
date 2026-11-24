---
name: council
description: Independent code review or critique across multiple providers, with an explicit single-provider fallback. Use when the user says "council", "council of 2", or "cheap council".
---

# Council

Independent review by strong models from different providers, with a clean fallback when only one provider is available. Fix known issues and run checks first so reviewers audit the real change.

## Seats

### Multi-provider (when available)

- **Default (council of 3)**: one latest flagship model per available provider (e.g. Claude, Codex, Antigravity) at high reasoning effort. In T3 Code, resolve models via `orchestrator_capabilities` and launch read-only delegates in parallel via `delegate_task`.
- **Council of 2 / cheap council**: two providers, excluding the provider that authored the change.
- If the user explicitly names a provider or model that is unavailable, report it instead of silently substituting.

### Single-provider fallback

When only one provider is available (and the user did not request a specific missing provider):
1. **State the fallback explicitly** before reviewing (never present a single-provider pass as a multi-provider council).
2. Launch **1–2 fresh, read-only subagents** (no author conversation history) with the brief below — using a distinct strong model if available on that provider, or splitting two independent passes (correctness & callers vs. scope & edge cases).
3. If the harness has no subagent tool, run one cold-read self-review against `git diff <base>` using the exact same rubric.

Never edit the checkout while reviewers are reading it.

## Code review brief

```text
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Read surrounding code. Find bugs, broken callers, mismatch with intent.
Skip style unless it hides a bug. Hold to the smallest change; flag unneeded code, tests, or guards.
blocker = intended case wrong. should = real bug callers will hit. nit = rest.
Reply:
SCORE: <n>/10
VERDICT: APPROVE or CHANGES
C1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion>
```

## Acting on findings

Verify every finding against the source yourself; re-classify rare cases or minor costs as nits.
- **Agree**: make the smallest fix (prefer deleting/narrowing code over adding guards or wrappers).
- **Disagree**: cite file:line, test, or output.
- **Out of scope / nit**: decline in one line and list as a follow-up.

## Rounds & report

- Budget: **2 rounds** (3 only for auth, security, data loss, or concurrency with an open blocker/should). Re-reviews use a fresh reviewer/delegate with the ledger of `fixed:`, `disagree:`, and `declined:` items.
- Stop early when a round returns only nits. If disputed after 2 replies or budget ends, present both sides in 2–3 lines for the user to decide.
- Report a compact score table per round (`no score` if omitted; never invent one) plus fixed items, disagreements, and follow-ups. Add to a PR description only if asked.
