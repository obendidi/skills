# skills

A 3-tier operating loop and companion review/PR skills for coding agents (**Claude Code**, **Codex**, **Antigravity**, **OpenCode**, **Cursor**, and **T3 Code**).

## Install

Requires Node.js 22.20+ (via [`skills`](https://github.com/vercel-labs/skills)).

**Global install across providers** (recommended):

```bash
npx skills add obendidi/skills -g -a claude-code -a codex -a antigravity -a opencode
```

*(Pass `-a antigravity` explicitly if you use Antigravity — the CLI's auto-detector checks `~/.gemini/antigravity` rather than `~/.gemini/config/skills`.)*

**Interactive or project-local install**:

```bash
npx skills add obendidi/skills
```

**Update installed skills later**:

```bash
npx skills update
```

## Skills

| Skill | Tier / Role | Triggers | What it does |
| :--- | :--- | :--- | :--- |
| [`quick`](skills/quick/SKILL.md) | **Tier 1** — Quick bite | *(default)*, `tier 1`, `quick` | Read callers, smallest complete diff, 1 targeted check, self-review, 1-line risk close |
| [`feature`](skills/feature/SKILL.md) | **Tier 2** — Involved feature | `tier 2`, `feature`, Plan mode | Read-only recon → align (≤2 rounds) → 1-page plan → approval gate → slice-by-slice build |
| [`lab`](skills/lab/SKILL.md) | **Tier 3** — Empirical research / hard bug | `tier 3`, `lab`, `debug`, `diagnose`, `spike` | Runnable harness first → isolated worktree experiments → independent verification → clean promotion |
| [`council`](skills/council/SKILL.md) | Review / critique | `council`, `council of 2`, `cheap council` | Independent multi-provider review (with explicit single-provider subagent fallback) |
| [`pr`](skills/pr/SKILL.md) | Pull requests | file, open, comment on, monitor, or babysit a PR | Clean PR descriptions, agent-attributed GitHub comments, and scope-disciplined babysitting |

## Recommended `AGENTS.md` Router (for Tier 1 by default)

Skills load on demand when triggered. To make Tier 1 your zero-overhead default on every small task without loading a skill file, paste this table into your global `AGENTS.md`, `CLAUDE.md`, or `GEMINI.md`:

```markdown
## Task Tiers

Match the workflow to the problem size. Suggest stepping up a tier in one line if scope surprises you, but never switch silently.

| Tier | Trigger | Workflow |
| :--- | :--- | :--- |
| **1 — Quick bite** | *(default — no keyword)* | Read callers, make the smallest complete diff (no speculative wrappers/options), run 1 targeted check (add a small test/assert for non-trivial logic), self-review the diff, and close with 1 line on what went unchecked or any risk. |
| **2 — Involved feature** | `tier 2`, `feature`, or Plan mode | Load the `feature` skill (recon → align → 1-page plan → approval gate → build). |
| **3 — Lab (research / hard bug)** | `tier 3`, `lab`, `debug`, `diagnose`, `spike` | Load the `lab` skill (runnable harness first → worktree experiments → verified promotion). |
```

## Multi-provider vs. Single-provider Harnesses

- **T3 Code / multi-provider setups**: `lab` and `council` fan out across available providers (e.g. Claude, Codex, Antigravity) using isolated worktrees and read-only delegates.
- **Single-provider CLIs (Claude Code, Codex CLI, OpenCode, Antigravity)**: `lab` runs isolated experiments via subagent worktrees or local `git worktree` checkouts, and `council` explicitly falls back to fresh, read-only subagents (or a cold-read diff review if subagents are unavailable) so every skill works standalone.

## License & Credits

MIT License — see [LICENSE](LICENSE).

- [`skills/council`](skills/council/SKILL.md) is adapted from [`t3dotgg/tstack`](https://github.com/t3dotgg/tstack) (MIT License — see [skills/council/LICENSE](skills/council/LICENSE) and [skills/council/SOURCE](skills/council/SOURCE)).
