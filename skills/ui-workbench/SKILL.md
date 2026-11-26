---
name: ui-workbench
description: Design, build, polish, and audit clean, utilitarian web apps, dashboards, data tools, and interfaces with dual-theme semantic tokens, progressive disclosure, disciplined typography, and crisp copy. Use when creating, polishing, auditing, or reshaping a web UI, React/Tailwind app, workbench, dashboard, or landing page.
---

# UI Workbench

Default to a calm, high-signal **workbench aesthetic** — crisp light surfaces, Monaco-inspired dark surfaces, semantic status tokens, progressive disclosure, and zero decorative clutter. When building a public brand or marketing page with an explicit visual brief, honor the brief's aesthetic while keeping the same restraint and craft.

## 1. Authority, Mode & Plan

Set the visual authority before changing code:
- **Preserve** (feature or polish in an existing app): inherit its tokens, shared components, and layout. Never smuggle a redesign into a polish or feature task.
- **Expand** (partial system): keep confirmed traits and assets; derive only what is missing.
- **Replace / Create** (approved redesign or greenfield): use the defaults below or the explicit visual brief.

Match the workflow to the ask:
- **New screen or redesign — two-pass plan (6–10 lines before coding):**
  - **Format & primary job**: Workspace/dashboard/tool (open directly into live content) or showcase/landing page (earned hero)? What single decision or action does the user come here to make?
  - **Tokens & surfaces**: Semantic base palette (light + dark), radius, and 1 deliberate accent.
  - **Layout & hierarchy**: Viewport shell, primary surface, and what secondary detail hides behind hover, expand, or click.
  - Critique the plan once before coding: cut any element that reads like a generic template default (arbitrary KPI strip, identical rounded SaaS card grid, decorative badges, or filler copy).
- **Polish / refinement — triage in order** (fix the cause at the narrowest level; don't perfect one corner while the main flow is broken):
  1. Broken or blocked tasks, data loss, misleading state, and inaccessible paths.
  2. Missing loading, empty, error, disabled, and recovery feedback.
  3. Flow, hierarchy, responsive breakage, and design-system drift (missing token vs. bypassed shared component vs. local bug).
  4. Visual, copy, and motion consistency.
- **UI audit (read-only unless fixes were requested):** walk the live task and report findings by severity (**P0** blocks task/risks data loss, **P1** major usability or a11y failure, **P2** friction with workaround, **P3** minor polish) without turning taste into defects. Separate confirmed runtime defects from source-level risks.

## 2. Layout & Opening Screen

- **Workspaces, dashboards, tools, docs, and data apps**: Never put a marketing hero banner above the workspace. Open directly into the live tool, table, map, or canvas.
  - Default app shell: full-viewport (`h-dvh min-w-0 flex-col`), collapsible icon-rail sidebar (`defaultOpen={false}` with tooltips when collapsed; footer holds theme toggle, user, and collapse trigger), compact single-line header with left/right action slots, scrollable `<main>`.
  - Make rows clickable for navigation/detail; put secondary actions behind hover, tooltips, or expandable panels instead of repeating buttons on every row.
  - Group sections with whitespace, `1px` hairline borders, or subtle surface tint — avoid nested card-in-card boxes and heavy drop shadows.
- **Public / hero-led pages (only when the brief calls for one)**: Skip the cliché 50/50 split (`H1 + paragraph + button + 3 stat columns` beside a media box). Let the headline or live visual command the viewport with ≤25 words of supporting copy and one primary action.

## 3. Dual-Theme Semantic Tokens

Define tokens once in CSS variables (`:root` + `.dark`) and map them via Tailwind `@theme inline` — never scatter raw palette classes (`bg-red-100`, `text-green-700`) across components. Set the `.dark` class in a tiny inline `<head>` script before first paint to prevent light/dark flash.

**Default neutral + workbench palette** *(override when a codebase or brief defines its own)*:
- **Light (`:root`)**: white canvas (`oklch(1 0 0)`), cool-slate ink (`oklch(0.129 0.042 264.695)`), soft muted surfaces (`oklch(0.968 0.007 247.896)`), hairline borders (`oklch(0.929 0.013 255.508)`), muted ink (`oklch(0.554 0.046 257.417)`), radius `0.625rem`.
- **Dark (`.dark`)**: editor-grade neutral dark — `#1E1E1E` canvas, `#252526` cards/sidebar/popovers, `#2A2D2E` hover/muted, `#454545` borders, `#3C3C3C` inputs, `#CCCCCC` foreground, `#CCCCCCB3` muted text, `#0E639C` primary action, `#007FD4` focus ring.
- **Semantic status contracts**: Every status (`success`, `warning`, `info`, `destructive`) ships paired solid (`--x` + `--x-foreground`) and subtle pill (`--x-subtle` + `--x-subtle-foreground` + `--x-border`) tokens. In dark mode, derive subtle surfaces with `color-mix(in srgb, <accent> 14%, var(--background))` and interactive row tints with `color-mix(in srgb, var(--foreground) 3%, transparent)`.
- **No AI color reflexes**: Do not reach for warm cream (`#F4F1EA`) + terracotta (`#D97757`) or near-black + neon purple/acid-green glow unless asked.

## 4. Typography, Controls & Motion

- **Typography**:
  - Use 1 versatile sans family (or at most 2 clearly distinct families). Do not add a 3rd monospace font for numbers, prices, or durations — use `tabular-nums` on the body font (reserve monospace for actual code, IDs, or logs).
  - Lock at most **3 body text sizes** below the page title (`4` only in dense multi-column tables); differentiate hierarchy with weight and muted color rather than extra pixel steps. Keep prose `<= 65–80ch` and never place `< 13px` text as primary above-the-fold copy.
  - Use **sentence case** for labels, tabs, and table headers — avoid tracked-out `ALL CAPS` eyebrows.
- **Controls & primitives**:
  - Prefer accessible primitives (Radix / shadcn) and a single crisp icon set (e.g. Lucide `size-4`).
  - Use non-button selection controls (segmented toggles, tabs, switches, `<select>`, combobox) for modes and filters rather than walls of bordered buttons.
  - Reserve rounded pills (`rounded-full px-2 py-0.5 text-xs font-medium`) for real semantic state (e.g. `completed`, `failed`, `running`) — never decorative tags, fake `"LIVE"` dots, or `01 / 02` prefixes on non-sequential headings.
- **Motion**:
  - Motion answers user action (opening, expanding, confirming), never idle decoration. Keep transitions brisk (`140ms–240ms`, ease-out), name explicit CSS properties (never `transition: all`), and honor `@media (prefers-reduced-motion: reduce)`.

## 5. UI Copy

- Every word in the UI must earn its place. Cut helper paragraphs, caveats, and onboarding slogans unless they resolve genuine ambiguity or prevent a real user mistake.
- Write from the user's perspective in active voice: `"Save changes"`, not `"Submit"`. Keep verbs consistent across the flow (`"Publish"` button → `"Published"` toast).
- Treat empty and error states as direction, not mood: state plainly what happened and the next action to fix it. No apologies, no vague errors, no `→` tacked onto buttons, and no em-dash clutter.
- Keep one language per surface.

## 6. Visual Verification & Audits

Before finishing, inspect the rendered UI in a browser when tools are available:
- Check both **light and dark themes** and **desktop + narrow/mobile (`390×844`)** viewports.
- **Visibility & reachability**: open real menus, popovers, dialogs, and scroll regions; confirm controls are not clipped, occluded by sticky bars/overlays, or unreachable by keyboard/scroll (DOM presence does not prove visibility).
- **Effective contrast**: verify body text (`>= 4.5:1`), large text (`>= 3:1`), controls, and focus rings against their actual rendered background (translucent layers, hover tints, gradients, images) rather than inferring from token names.
- **Layout & numbers**: confirm single-line headers, aligned numeric columns (`tabular-nums`), visible keyboard focus, zero horizontal overflow, and zero clipped text.
- **State & form recovery**: exercise reachable loading, empty, error, and form recovery paths (invalid input → fix → blur/submit → leave and revisit) so stale error markers do not linger.
- **Honest coverage**: separate code checks from rendered inspection, and state any unchecked themes, viewports, or states explicitly.
