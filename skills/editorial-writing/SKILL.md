---
name: editorial-writing
description: Write human-friendly technical blog posts, engineering notes, and launch articles that lead with a clear story in ordinary language and keep heavy technical reference in the README or expandable notes. Use whenever drafting, structuring, or rewriting a technical blog post, dev.to/Substack/company engineering article, or public project launch write-up where readers should understand the problem and why it matters before seeing charts, benchmarks, or code.
---

# Editorial Technical Writing

Write technical articles that read like a clear story from a builder, not a condensed README or benchmark dump. Readers should understand the problem in plain terms and know why they should care before they see a chart or a command.

Use `plain-writing` for PRs, RFCs, READMEs, and changelogs where dense technical precision is the point. Use this skill for blog posts, launch write-ups, and engineering notes meant to be read end to end, and pair the final draft with `humanizer` to strip AI writing tells.

## Core Principles

### 1. Story first, reference in the repo
A blog post's job is to explain the problem, the approach, and the real-world result. Heavy reference material belongs in the repo README, migration guides, or collapsed `<details>` notes, linked once from the story:
- Keep out of the blog post: full model or parameter matrices, flag encyclopedias, column-by-column schema tables, multi-screen glue code, and secondary ablation tables.
- Keep in the blog post: the concrete problem, why existing options fall short, the core idea, the main visual or benchmark chart, and direct links to the README or scripts for readers who want to reproduce or go deeper.

### 2. One rhythm per section
Each section should answer a simple human question before asking the reader to read a chart or run a command:
1. **Frame the problem in everyday terms** (what breaks, what costs too much, or what a user experiences).
2. **State the approach and takeaway in 2 to 4 plain sentences** so the reader knows what to look for.
3. **Show the one visual or tiny command** (GIF, Pareto frontier, comparison chart, or 1 to 3 line snippet) that proves the point.
4. **Point to the repo or expandable notes** for exact tables, hardware setups, and reproduction steps.

Never drop a chart without first explaining the metric in plain English, and never describe the chart's layout ("the chart below shows...") instead of stating the finding directly.

### 3. Rounded prose, exact evidence
Numbers in narrative sentences should be easy to read aloud; exact numbers live in charts, code, and linked tables:
- Round narrative estimates to 1 or 2 significant digits (*"nearly 60,000 frames"*, *"about a second"*, *"close to 10 frames per second"*, *"roughly $7,300 a month"*). Never round across a threshold that changes the conclusion.
- Keep dates, published prices, versions, meaningful thresholds, and reproducible CLI flags exact.
- Name the measurement's scope honestly (live vs. offline replay, laptop CPU vs. GPU, general-purpose model vs. dataset-specific academic SOTA). Keep caveats that change how a reader interprets the chart right next to the chart.

### 4. Plain English over jargon walls
Read the title, opening paragraph, and every section lead aloud. If a capable engineer outside your sub-specialty trips over acronyms or checkpoint names, translate them into what the thing does:
- **Title**: lead with the reader's problem or outcome, not a list of acronyms or licenses.
- **Code budget**: at most 1 or 2 short commands that a reader can grasp in three seconds. Move full APIs and SDK migrations to linked docs.
- **No acronym piles**: introduce a tool by what it lets you do, not by listing five model IDs and four algorithm names in sentence one.
- **Respect the reader's judgment**: show real benchmarks, honest context, and clear tradeoffs so readers decide for themselves whether the tool fits, rather than lecturing them with boilerplate "When NOT to use this" checklists (keep real safety or compatibility caveats when they prevent silent mistakes).

## Before / After Examples

**Opening with jargon vs. plain story**
- *Before:* "I built ToolX (v0.6.0) around Apache-2.0 ONNX Runtime detectors (`rtdetr-v2-r18`, `dfine-n/s/m`) and four multi-object trackers (ByteTrack, BoT-SORT, OC-SORT, BoostTrack++) exposed via CLI and streaming API."
- *After:* "I built ToolX so you can run people and object tracking on a laptop or local server, without setting up a heavy surveillance stack or paying for a commercial license."

**Statistic dump vs. rounded prose + evidence**
- *Before:* "Across 59,812 frames on 4 splits, median latency is 0.98 s and mean HOTA reaches 38.8 vs 37.5 for baseline at 2,072 tracker FPS and 9.4 end-to-end FPS."
- *After:* "Tested on nearly 60,000 frames across four public benchmarks, it leads on accuracy while running close to 10 frames per second on a laptop. Full per-dataset numbers and the reproduction script are in the README."

**Explaining a metric before a chart**
- *Before:* "Here is the HOTA Pareto frontier across DanceTrack, SportsMOT, MOT17, and MOT20."
- *After:* "Drawing a box around a person in a single frame is simple. Keeping the same ID on that person when crowds cross paths or a camera pans is where trackers break down. Every tracker in the benchmark received the exact same detections from one default model, so the comparison isolates the tracking algorithms."

## Final Check Before Publishing

1. **Humanizer pass**: run the `humanizer` skill (if installed) over the finished prose to strip AI tells (dashes kept only at the writer's sample rate, not-X-but-Y staging, bold-colon lists, stock AI words, one-line closers) while preserving the writer's voice if a sample was provided.
2. **README offload check**: scan every table, code block, and bullet list. If a detail is reference material rather than part of the story, replace it with one plain sentence and a link to the README or docs.
3. **Truth and link check**: verify every rounded narrative claim matches the exact chart/source data, and verify every linked doc and embedded image URL resolves.
