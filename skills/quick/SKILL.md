---
name: quick
description: Tier 1 workflow for small, well-scoped tasks and fixes. Use when the user explicitly says "tier 1" or "quick", or when no global AGENTS.md Tier 1 loop is loaded.
---

# Quick (Tier 1)

Default loop for small, localized edits where the shape is already clear.

1. **Read callers first**: inspect the target code, its callers, and nearby tests before editing.
2. **Smallest complete diff**: fix the root cause with no speculative wrappers, options, or unrelated cleanup.
3. **Verify**: run every cheap local check needed to confirm end-to-end behavior (add a small test or assert for non-trivial logic) — don't stop after a partial check and punt an easy probe to the user.
4. **Self-review**: re-read your diff and cut dead code, extra guards, or scope creep.
5. **Close cleanly**: note remaining risk or unchecked gaps only when a check is genuinely blocked, slow, paid, or external.

## Escalating

If scope surprises you (contract or schema change, multi-service blast radius, or an unverified root cause), suggest stepping up to `feature` (Tier 2) or `lab` (Tier 3) in one line — never switch silently.
