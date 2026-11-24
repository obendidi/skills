---
name: feature
description: Tier 2 workflow for involved features, refactors, or multi-service/high-impact changes. Use when the user says "tier 2" or "feature", or in Plan mode.
---

# Feature (Tier 2)

For refactors, multi-service work, contract/schema changes, or features where getting the shape right matters before writing code. Stay read-only until the plan is approved.

## 1. Recon (read-only)

Read the affected modules, callers, contracts, tests, and repo conventions before asking anything. Don't ask questions the codebase already answers.

## 2. Align (max 2 rounds)

Only ask what materially changes what gets built:
- Batch at most 5 numbered questions per round, each with your recommended default.
- Never drip questions one at a time.
- If the user says "just build" or skips alignment, state your key assumptions in ≤3 bullets and drop to Tier 1 (`quick`).

## 3. Plan (≤40 lines in chat)

Write a concise plan in chat (no committed spec file unless asked):
- **Goal & non-goals**: what changes and what stays out of scope.
- **Surfaces & contracts**: exact files, APIs, schemas, or migrations touched (and owners if cross-team).
- **Tradeoffs**: 2–3 viable options only when a real choice exists, with your recommendation.
- **Slices**: ordered implementation steps, each with its concrete verification command.
- **Risks & rollback**: failure modes, data/compat hazards, and how to revert safely.

Use visual diagrams or inline previews (e.g. T3 `html_render` or Mermaid) when seeing beats reading.

## 4. Approval gate

Stop and wait for explicit user approval (or native Plan mode approval) before editing product code, installing dependencies, or scaffolding.

## 5. Build & verify

- Implement slice by slice, running each slice's verification command before moving on.
- **Drift stop**: if implementation uncovers an unplanned contract change, migration, or wide blast radius, stop and re-plan. After 2 re-plans, pause for the user.

## 6. Review & ship

- Run a smallest-diff self-review pass on the final diff (cut dead code, unneeded wrappers, and scope creep).
- For high-impact changes (auth, data, contracts, multi-service), offer a 2-seat `council` review on the final diff.
- Ship through the `pr` skill only when asked.
