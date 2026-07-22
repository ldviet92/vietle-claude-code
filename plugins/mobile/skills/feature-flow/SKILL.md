---
name: feature-flow
description: Orchestrate a multi-screen feature in a React Native app via a 5-step playbook — each step produces a checked-in HTML artifact under design/<feature>/ (context → flow-mockup → api → plan → execute). Use when adding any non-trivial multi-screen feature (3+ screens, or feature with its own tab/nav root). Gates between steps for user approval. Composes brainstorming + new-screen + code-reviewer.
---

# Feature flow

End-to-end playbook for shipping a new multi-screen feature the way it produces clean phased PRs: align WHO/WHY first, lock the visual flow, lock the API surface, lock the plan, then execute one phase at a time.

## When to activate

- User asks to "build a new feature" / "add the X feature" / "design and build X"
- Feature spans 3+ screens, or has its own tab/nav root, or its own backend surface
- User explicitly types `/feature-flow <name>`

## Do NOT use for

- Single-screen work → use `new-screen` directly
- Cloning/porting an existing screen or panel → use `clone-screen` (port-phase work locks scope/fields/validation, no brainstorm needed)
- Bug fixes / refactors / pure visual tweaks
- Documentation-only changes

## Hard rules

1. **Gates between every step.** Do not start step N until the user has approved the artifact from step N-1. Skipping a gate is what produces wrong-thing-built.
2. **Every step writes an HTML artifact under `design/<feature>/`** that gets checked in. The artifacts are the spec — they outlive the PR.
3. **Templates are canonical.** Mirror the structure of `design/cleaning/context.html` and `design/cleaning/plan.html` (in the dayladau app-host repo). Don't reinvent the layout.
4. **Ionicons via CDN, never emoji** in mockup files (`<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ionicons@7/dist/css/ionicons.min.css">` + `<ion-icon name="...">`). This keeps RN translation 1-to-1.

## The 5 steps

### Step 1 — Brainstorm → `design/<feature>/context.html`

**Goal:** lock WHO + WHY before any pixels.

If the `brainstorming` skill is installed, invoke it now to surface the WHO/WHY. Capture its output into `context.html`.

Required sections (mirror `design/cleaning/context.html`):
- **Eyebrow + h1 + lede** — one-sentence framing of the mental model
- **TOC pills** — anchor links to each section
- **§1 Who this is for** — persona blocks with avatar tile + name + role + 2–3 sentences on context (time-poor? interrupted? what they're trying to accomplish)
- **§2 What they actually care about** — the questions in their head when they open this screen. Followed by a "Rule of thumb" tinted card
- **§3 Core principles** — numbered (1, 2, 3, …), each with a one-line h4 + one-sentence explanation
- **§4 Anti-patterns** — a two-column before/after (`.ba`) block contrasting old approach vs new. Followed by a `.card.bad` list of specific things to avoid
- **§5 The flow at a glance** — `<ol>` of every screen with a one-line purpose. Include status pills where relevant (e.g., `<span class="pill bad">2 trễ</span>`)
- **Footer** — line on when to update this doc ("update when the user model shifts, not when you tweak a card border")

**Gate.** Present a 5-line summary of context.html (personas + #principles + #anti-patterns + #screens) and wait for user approval before step 2.

### Step 2 — Design flow → `design/<feature>/flow-mockup.html`

**Goal:** lock the visual layout per screen.

Build a static HTML mockup of each screen in the flow. One column per screen, captions describing the transition between them. Use Ionicons via CDN (never emoji). Mirror project design tokens (`--brand`, `--text`, etc. from `docs/Design.md`).

For multi-role features, write one mockup per role (e.g., `flow-mockup.html` for host, `flow-mockup-cleaner.html` for cleaner).

**Gate.** Show the user the mockup file path and wait for approval before step 3.

### Step 3 — API mockup → `design/<feature>/api.html`

**Goal:** lock fields ↔ endpoints before any screen code.

For each screen in `flow-mockup.html`, document:
- Endpoint(s) called on mount / on action
- Request shape (params, body)
- Response shape (the subset the screen actually uses)
- Any new endpoints needed vs reused existing ones (link to existing service files)
- Field-level notes: defaults, validation, format quirks (e.g. time format `2026-05-03T1400`)

For multi-role features, write one per role.

**Gate.** Show user the api.html and wait for approval before step 4.

### Step 4 — Plan with embedded tracker → `design/<feature>/plan.html`

**Goal:** turn the locked spec into a phased execution checklist.

Mirror the structure of `design/cleaning/plan.html`:
- **Overall progress card** — `N / N tasks`, progress bar
- **Per-phase cards** — each with phase number tile, h2 title, one-line meta, status pill (`Pending` → `In progress` → `Done`), per-phase progress (`X / Y tasks`)
- **Per-task checkboxes** — each `<li>` has the task description + a tool tag pill (`Edit` / `Write` / `Skill` / `Agent`)
- **Workflow-rules box** at the bottom restating the per-phase process (flip pill → `/new-screen` for screen tasks → direct Edit/Write for non-screen → `code-reviewer` on phase diff → tick reviewer item → next phase)
- **Companion-files footer** — links to context.html, flow-mockup*.html, api*.html

Phase 0 should always be **Foundation** — types, services, store skeleton, nav scaffold (plumbing, no screens). Subsequent phases are per-role or per-user-journey, each ending in a `code-reviewer` task.

**Gate.** Show user the plan.html and wait for approval before step 5.

### Step 5 — Execute each phase

For each phase in plan.html, follow the embedded workflow rules:

1. Flip the phase pill `Pending` → `In progress` in plan.html
2. For each task in the phase:
   - **Screen tasks** (those tagged `Skill`) → invoke the `new-screen` skill
   - **Non-screen tasks** (those tagged `Edit` / `Write`) → use Edit/Write directly following project conventions
3. When all tasks in the phase are done, invoke the `code-reviewer` agent on the **whole phase diff** (not per-file)
4. Address every flag from the reviewer before declaring the phase done
5. Tick the reviewer item, flip phase pill to `Done`, update overall progress numbers
6. Surface a one-line summary to the user (what shipped, what flags were addressed) — then move to next phase

**Important:** Update `plan.html` as you go. Do not batch updates at the end of the phase.

## Compose with project agents

If the target project has worker agents under `.claude/agents/`, prefer them over inline work in step 5:
- `api-integrator` — when a phase touches types + services + store
- `screen-builder` — when a phase is one or more screens
- `code-reviewer` — at the end of every phase

These already encode project conventions. Read them in step 4 when sizing tasks so the plan reflects who/what does each task.

## What this skill does NOT do

- Does not commit, push, or open PRs — wait for the user to ask
- Does not run the app on a device — remind the user to do that at the end of each phase
- Does not create Linear tickets — separate ask
- Does not pick the feature name — ask the user when activated without arguments

## Output

When activated, restate the feature name and the 5 artifact paths it will produce, then start step 1. Between every step, present the artifact path and a one-paragraph summary, then explicitly ask: *"Approve to move to step N+1?"*

When all 5 steps are done, surface a 3-line wrap:
- What shipped (feature name + N phases + screens count)
- What's left for the user (manual device test, commit, PR, Linear ticket)
- Where the spec lives (`design/<feature>/`)
