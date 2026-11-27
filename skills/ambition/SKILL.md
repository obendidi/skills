---
name: ambition
description: Anchored pre-flight reframing to sharpen a brief before building or researching, using Altitude, Depth, and Range without scope creep. Use when the user says "ambition", "more ambitious", "ambition check", "reframe this brief", "push further", "deepen this brief", or "goal beyond the goal", or wants to check whether a task or research goal is aimed too low before running feature, lab, or deep lab.
---

# Ambition (Anchored Reframe)

Raise the ceiling of a task, spec, or research goal **without going wild across unrelated dimensions**. Read-only: never edit files or expand scope until the user picks a dial.

## 1. Anchor first (stay grounded)

Before proposing anything bigger, read any relevant local context and state in ≤4 lines:
- **Real outcome**: what decision, capability, or measurable result the user actually wants.
- **Hard constraints**: fixed deadlines, budget (`$0` compute unless stated), must-keep interfaces, or explicit non-goals.
- **Current frame**: the task as stated today.

If the real outcome or a hard constraint is unclear, ask **1 batched round** (≤3 numbered questions, each with a recommended default) and wait before reframing.

## 2. Diagnose along 3 lenses (pick only what fits)

Evaluate each lens against the anchor. **Skip any lens that does not serve the real outcome** (state `Skip — <reason>` rather than inventing scope):

1. **Altitude (goal beyond the goal)** — *Skip for fixed deliverables or creative work.* Ask what happens *after* this task and whether a higher-leverage loop or system beats a one-off artifact (e.g. a durable pipeline or backtested early-warning check instead of a static report). Note any new workflow/data access it would require.
2. **Depth (expert layer, not surface summary)** — Run the 4-part check:
   - What would a top specialist do here that the current brief isn't asking for?
   - What is the **rawest source material** (logs, traces, raw tables, ground truth) instead of pre-digested summaries?
   - What **real field instruments** (database, statistical model, profiler, simulator, solver) replace prose or toy heuristics?
   - What **outside judge** (executable test, holdout data, replay harness — never self-grading) proves the result is right, and what is the **hard physical/theoretical target** instead of a timid tweak?
3. **Range (adjacent fields by method, never roleplay)** — Strip the domain noun down to its core **verb** (what mechanism is actually being performed under what constraints). Name 1–2 unrelated fields that solve that verb at scale and extract the **concrete technique or red-team check** they run. Never ask for persona roleplay ("write like X"); import what X actually computes or verifies.

## 3. Let the user pick the dial (hard stop)

Present a compact menu (≤4 options) — always including **Keep current scope** — with each option naming **which lens(es) turn up**, **expected payoff**, and **extra cost/risk**. Put your single recommended option first and **stop**.

## 4. Lock the brief & hand off

Once the user chooses, output one concise **Reframed Brief** (goal & stretch target, rawest inputs & instruments, outside judge, imported cross-field method if chosen, and explicit out-of-scope guardrails). Then hand off cleanly:
- Open design branches still unsettled → suggest `grill-me`
- Implementation / architecture work → suggest `feature`
- Empirical optimization / research → suggest `lab` or `deep lab`
