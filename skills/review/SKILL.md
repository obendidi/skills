---
name: review
description: Quality, security, elegance, and value review of a code change, commit, branch, or pull request. Checks correctness and spec alignment, whether the change is worth it and tracks adoption, design elegance vs. brute-force hotfix, high-confidence security and data safety, scale, tests, and bloat. Use when the user asks to "review this", "code review", "review my PR", "review the last commit", "is this over-engineered", or "is there a cleaner way".
---

# Review

Read-only review of a change from the perspective of the engineer who will own it six months from now. Do not edit, commit, push, or post PR comments unless explicitly asked.

## 1. Understand First

- **Scope**: Review what the user named (uncommitted diff, commit, branch, PR, or files). If nothing was named, review uncommitted changes, or `HEAD~1..HEAD` if clean.
- **Read beyond the diff**: Inspect callers of every changed function, the functions it calls, nearby tests, and repo rules (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`). A change can break untouched callers.
- **Identify intent / spec**: Check the PR description, commit messages, or linked issue/plan so you know what problem the change is supposed to solve.
- **Calibrate load**: Note the expected scale (single-user script vs. multi-user/concurrent service) and judge scale findings against it.

## 2. Review Dimensions (in Priority Order)

1. **Correctness & spec alignment**: Bugs, broken callers, unhandled edge cases (empty, zero, timezone, partial failure), requirements from the spec/issue that are missing or wrong, and unintended scope creep.
2. **Value & adoption (is it worth it?)**: Does this actually solve the root problem, or is it solving the wrong/low-value problem? If it introduces a new user-facing surface, workflow, or tool, can adoption and real usage be observed (logs, metrics, events), or is it shipping blind?
3. **Security & data safety (high-confidence only)**: Trace data flow across the codebase before flagging. Distinguish **attacker-controlled input** (request params, headers, cookies, uploads, URL segments, other users' data) from **server-controlled config** (env vars, settings, hardcoded constants) and framework mitigations (ORM parameterization, auto-escaping). Flag exploitable injection, auth/authz bypass, SSRF, secrets, or data-loss hazards (swallowed errors, wrong write order, missing transactions). Skip theoretical warnings and test-only noise.
4. **Design elegance vs. hotfix**: Ask whether this is an ugly band-aid (special-case flags, duplicated workarounds, patching symptoms at the wrong layer) or a clean design that simplifies the system and shrinks complexity. When a cleaner redesign exists, sketch it concretely so the author can weigh it. Pragmatic brute-force fixes are acceptable when time-boxed or when no cleaner seam exists — label them honestly as tactical trade-offs rather than blocking for aesthetics.
5. **Scale & tests**: Concurrency races (check-then-write), N+1 queries, unbounded memory growth, or O(n²) hot paths at expected load. Flag risky new logic (branches, parsers, money, auth, writes, bug fixes) lacking one test that fails when it breaks — not coverage percentages.
6. **Lean / anti-bloat**: Code that should be deleted or shrunk — dead code, unused options, speculative abstractions with one implementation, helpers reinventing the repo or standard library, or new dependencies added for a few lines.

## 3. Noise Gate (Check Before Reporting)

- Every finding needs a **concrete failure case, verified unused/duplicated path, or concrete simpler alternative** — no vague worries or "consider X".
- Skip style, formatting, or anything linters/typecheckers already enforce unless it hides a real bug.
- Re-read the referenced lines to verify the caller exists, the input can really be attacker-controlled or empty, or the code is genuinely unused.
- **Impact**: Tag each finding `observed` (logs, tickets, hit in normal use), `likely`, or `rare` (needs unusual timing or sequence). A `rare` finding outside auth, security, or irreversible data loss is a nit.
- Propose the smallest fix that solves the cause. Prefer fixes that delete or simplify code over adding layers or guards.

## 4. Output Format

Start with **What this change does** (2–3 plain sentences, including whether it solves the stated problem and whether it reads as a clean fix or a tactical hotfix).

Then list findings in three groups (skip empty groups), numbered continuously across all groups so the user can say *"fix 1 and 4"*:
- **Blocker** (`blocker`): intended case wrong, exploitable security bug, data loss, or breaks at expected load.
- **Should fix** (`should`): real bug callers will hit, missing spec requirement, ugly hotfix with a clearly better seam, risky code without a test, adoption blindness on a major new surface, or clear bloat/duplication.
- **Nit** (`nit`): small simplifications, minor speed-ups, or optional polish.

For each finding, state in plain English:
`N. **Short title** (file:line) [observed|likely|rare] — What breaks or why it matters. Fix: smallest concrete change (or cleaner alternative).`

Close with one line: **Verdict** (`APPROVE` or `CHANGES`) and what went unchecked (e.g. runtime tests, live infra, missing spec).
