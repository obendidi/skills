---
name: plain-writing
description: Write clear, understated, concrete technical prose for PRs, RFCs, READMEs, changelogs, launch posts, and engineering updates without AI filler or marketing fluff. Use when drafting or editing technical writing, PR descriptions, docs, or release notes, or when the user asks to "deslop" or tighten text.
---

# Plain Technical Writing

Turn verbose or AI-sounding technical text into direct, verifiable prose that engineers actually read.

## Core Rules

1. **Conclusion first**: open with the verdict, problem, or user-visible outcome in one sentence. End with the next step or open decision. Never open a PR or doc with a file-by-file inventory.
2. **Concrete over vague**: name the exact module, metric, baseline, and sample size. Replace "significantly faster" with `78 ms -> 35 ms on 120 MOT17-09 frames`.
3. **Separate evidence from opinion**: keep observed facts, proposals, accepted decisions, and verified results distinct. State what went unchecked or is `unknown` instead of smoothing over gaps.
4. **Describe what it does now**: code comments, READMEs, and reference docs describe current behavior — keep conversation history out of durable docs, and put implementation history in commit/PR messages. Exception: RFCs and architecture decision records (ADRs) *must* preserve decision rationale, tradeoffs, and rejected alternatives.
5. **Match scope to the reader**:
   - **Changelogs**: user-visible outcomes only (no internal refactors unless behavior changed).
   - **Launch posts / READMEs**: state real limits and non-goals upfront — credibility comes from saying when *not* to use the tool.
   - **Drafts in the user's name**: match their cadence and keep only the main points.

## Cut AI Tells on Sight

Delete or rewrite these patterns before returning any draft:

- **Filler adverbs**: `just`, `really`, `very`, `simply`, `actually`, `basically`, `seamlessly`, `effortlessly`.
- **Marketing adjectives**: `powerful`, `robust`, `blazing-fast`, `easy`, `quick`, `modern`, `cutting-edge`, `game-changing`, `comprehensive`, `production-grade` (prove it with a number or delete it).
- **Vague quantifiers**: `significantly`, `substantially`, `many`, `various`, `numerous`.
- **Recap transitions & throat-clearing**: `In summary,`, `It's worth noting that`, `Let's dive into`, `Here's why that matters:`.
- **Punctuation & rhythm tics**: em-dash piles (`— ... —`), rhetorical "Not just X — it's Y" contrasts, and rule-of-three buzzword triads.

## Quick Self-Check Pass

Before finishing any technical doc, PR, or post:
- Can the first paragraph be cut without losing the main point? If yes, cut it.
- Does every claim have a concrete subject, action, or number?
- Are caveats and unmeasured pieces stated plainly instead of hidden?
