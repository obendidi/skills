# skills

[![skills.sh](https://img.shields.io/badge/skills.sh-obendidi%2Fskills-0f172a?logo=vercel)](https://skills.sh/obendidi/skills)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A lightweight operating loop (`ambition` pre-flight, 3-tier execution `quick` / `feature` / `lab`, plus `review`, `council`, and `pr`) and focused specialist skills (`local-llm`, `ui-workbench`, `plain-writing`, `editorial-writing`) for coding agents (**Claude Code**, **Codex**, **Antigravity**, **OpenCode**, **Cursor**, and **T3 Code**).

## Why this

Most agent skill packs skew either toward heavy end-to-end process on every edit (e.g., [`obra/superpowers`](https://github.com/obra/superpowers)) or PR/CI automation alone (e.g., [`t3dotgg/tstack`](https://github.com/t3dotgg/tstack)). Every workflow `SKILL.md` stays under 75 lines, built around two ideas:

- **Match ceremony to risk**: small fixes run with zero planning overhead (`quick`), multi-file changes gate on a ≤40-line plan (`feature`), and hard bugs require a runnable harness plus isolated worktree experiments before touching `main` (`lab`). Optional `ambition` reframes a brief before locking scope.
- **Multi-provider when available, standalone everywhere**: `lab` and `council` fan out across providers in T3 Code, and explicitly fall back to fresh read-only subagents or local `git worktree` runs in single-provider CLIs.

## Install

Requires Node.js 22.20+ (via [`skills`](https://github.com/vercel-labs/skills)).

**Global install across providers** (recommended):

```bash
npx skills add obendidi/skills -g -a claude-code -a codex -a antigravity -a cursor -a opencode
```

*(Pass `-a antigravity` explicitly if you use Antigravity — the CLI's auto-detector checks `~/.gemini/antigravity` rather than `~/.gemini/config/skills`.)*

**Interactive or selective install** *(for selective installs, pair `lab` with `council` + `review`, and `council` with `review` — each works standalone with inline fallbacks, but they compose best together)*:

```bash
npx skills add obendidi/skills
npx skills add obendidi/skills --skill lab --skill council --skill review
```

**Update installed skills later**:

```bash
npx skills update
```

## Skills

| Skill | Tier / Role | Triggers | What it does |
| :--- | :--- | :--- | :--- |
| [`ambition`](skills/ambition/SKILL.md) | **Pre-flight** — Anchored reframe | `ambition`, `more ambitious`, `ambition check`, `goal beyond the goal` | Anchor outcome & constraints first, diagnose Altitude/Depth/Range, let user pick a dial, and lock a sharper brief for `feature`/`lab` |
| [`quick`](skills/quick/SKILL.md) | **Tier 1** — Quick bite | *(default)*, `tier 1`, `quick` | Read callers, smallest complete diff, end-to-end local checks, self-review, risk/blocked-gap close |
| [`feature`](skills/feature/SKILL.md) | **Tier 2** — Involved feature | `tier 2`, `feature`, Plan mode | Read-only recon → align (≤2 rounds) → 1-page plan → approval gate → slice-by-slice build |
| [`lab`](skills/lab/SKILL.md) | **Tier 3** — Empirical research / hard bug | `tier 3`, `lab`, `deep lab`, `autoresearch`, `research loop`, `overnight lab`, `spike` | Runnable harness first → isolated worktree experiments (plus multi-round literature & frontier loop in [`deep lab`](skills/lab/references/deep-loop.md)) → independent verification → clean promotion |
| [`review`](skills/review/SKILL.md) | Single-pass code review | `review this`, `code review`, `review my PR`, `is this over-engineered` | Correctness & spec, value & adoption, high-confidence security, elegance vs. hotfix, scale/tests, and anti-bloat |
| [`council`](skills/council/SKILL.md) | Multi-provider review / critique | `council`, `council of 2`, `fast council`, `cheap council` | Independent multi-provider review using the `review` rubric (plus `fast council`, RFC/plan critique, and split-lens single-provider fallback) |
| [`pr`](skills/pr/SKILL.md) | Pull requests | file, open, comment on, monitor, or babysit a PR | Clean PR descriptions, agent-attributed GitHub comments, and scope-disciplined babysitting |

### Specialist skills

| Skill | Domain | What it does |
| :--- | :--- | :--- |
| [`local-llm`](skills/local-llm/SKILL.md) | Local Ollama / OpenAI-compatible CLI | Bundled zero-setup `uv` script for private OCR/vision, bulk classification, and strict JSON extraction (loopback-locked by default; MLX HTTP-501 structured-output fallback) |
| [`ui-workbench`](skills/ui-workbench/SKILL.md) | Web UI & dashboards | Calm workbench-first design system with dual-theme semantic tokens, progressive disclosure, disciplined typography, and visual-audit gates |
| [`plain-writing`](skills/plain-writing/SKILL.md) | Technical reference prose | Conclusion-first PRs, RFCs, READMEs, and changelogs with concrete numbers and AI-filler cuts |
| [`editorial-writing`](skills/editorial-writing/SKILL.md) | Technical blog & launch articles | Story-first engineering posts and launch write-ups in plain language, keeping heavy reference tables in the README |

## Recommended `AGENTS.md` Router (for Tier 1 by default)

Skills load on demand when triggered. To make Tier 1 your zero-overhead default on every small task without loading a skill file, paste this table into your global `AGENTS.md`, `CLAUDE.md`, or `GEMINI.md`:

```markdown
## Task Tiers

Match the workflow to the problem size. Suggest stepping up a tier in one line if scope surprises you, but never switch silently. Plan mode and explicit read-only scope override all tiers (allow read-only recon/profiling only; gate writes and experiments on approval).

| Tier | Trigger | Workflow |
| :--- | :--- | :--- |
| **1 — Quick bite** | *(default — no keyword)* | Read callers, make the smallest complete diff (no speculative wrappers/options), run every cheap local check needed to verify end-to-end behavior (add a small test/assert for non-trivial logic), self-review the diff, and note remaining risk or unchecked gaps only when genuinely blocked, slow, paid, or external. |
| **2 — Involved feature** | `tier 2`, `feature`, or Plan mode | Load the `feature` skill (recon → align → 1-page plan → approval gate → build). |
| **3 — Lab (research / hard bug)** | `tier 3`, `lab`, `deep lab`, `autoresearch`, `research loop`, `overnight lab`, `spike` (or `debug`/`diagnose` when the root cause stays unclear after a quick check) | Load the `lab` skill (runnable harness first → multi-provider worktree experiments or `deep lab` frontier loop → verified promotion). |
```

## Multi-provider vs. Single-provider Harnesses

- **T3 Code / multi-provider setups**: `lab` and `council` fan out across available providers (e.g. Claude, Codex, Antigravity) using isolated worktrees and read-only delegates.
- **Single-provider CLIs (Claude Code, Codex CLI, Cursor, OpenCode, Antigravity)**: `lab` runs isolated experiments via subagent worktrees or local `git worktree` checkouts, and `council` explicitly falls back to fresh, read-only subagents (or a cold-read diff review if subagents are unavailable) so every skill works standalone.

## Example: `lab` on a real regression

When [`obendidi/trackflow`](https://github.com/obendidi/trackflow) trailed Roboflow `trackers` 2.7.0 on MOT17 with YOLO26 Nano (`29.9` vs `32.9` HOTA), running `lab` forced a runnable evaluation harness first, then compared preprocessing hypotheses in isolated worktrees against the same-day control ([trackflow#117](https://github.com/obendidi/trackflow/pull/117)):

| # | Hypothesis | Change | MOT17 val HOTA (control: 29.9) | Verdict |
| :-: | :--- | :--- | ---: | :--- |
| 1 | Stretched 1920×1080 frames distort YOLO26 boxes | Letterbox to 640×640 with gray (114) padding | **37.4** (+7.5, beats `trackers` 32.9) | Promote |
| 2 | Apply same letterbox to default RT-DETRv2 R18 | Letterbox RT-DETRv2 inputs | 45.2 vs 45.1 (MOTA −1.2) | Discard |

Instead of debating preprocessing or changing every detector globally, the loop isolated the fix to the five YOLO26 presets, verified web-playground parity, and left DETR models untouched.

## License & Credits

MIT License — see [LICENSE](LICENSE).

- [`skills/council`](skills/council/SKILL.md) is adapted from [`t3dotgg/tstack`](https://github.com/t3dotgg/tstack) (MIT License — see [skills/council/LICENSE](skills/council/LICENSE) and [skills/council/SOURCE](skills/council/SOURCE)).
