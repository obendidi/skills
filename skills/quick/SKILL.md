---
name: quick
description: Tier 1 workflow for small, well-scoped tasks and fixes. Use when the user says "tier 1" or "quick", or as the default loop when no larger workflow is needed.
---

# Quick (Tier 1)

Default loop for small, localized edits where the shape is already clear.

1. **Read callers first**: inspect the target code, its callers, and nearby tests before editing.
2. **Smallest complete diff**: fix the root cause with no speculative wrappers, options, or unrelated cleanup.
3. **Verify**: run one targeted check (add a small test or assert for non-trivial logic).
4. **Self-review**: re-read your diff and cut dead code, extra guards, or scope creep.
5. **Close cleanly**: end with one line stating what went unchecked or any remaining risk.

## Escalating

If scope surprises you (contract or schema change, multi-service blast radius, or an unverified root cause), suggest stepping up to `feature` (Tier 2) or `lab` (Tier 3) in one line — never switch silently.
