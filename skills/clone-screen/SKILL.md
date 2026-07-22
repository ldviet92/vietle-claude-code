---
name: clone-screen
description: Port a screen from one app/codebase to another (e.g. web host → app-host, or app-guest → app-host) with alignment on what carries over, logic-layer tests, and project conventions. Orchestrates grill-with-docs → zoom-out (source + target) → tdd → screen-builder → code-reviewer.
---

# Clone Screen

End-to-end flow for porting a screen across apps or platforms. A clone is NEVER a 1:1 copy — platforms differ, conventions differ, scope shifts. Align first.

## When to activate

- User asks to clone, port, mirror, or re-create a screen from one app to another
- Web → mobile ports (e.g. `host/` → `app-host/`)
- Cross-app mobile ports (e.g. `app-guest/` → `app-host/`)
- User says "build the same X from Y in Z"

Do NOT use this for:
- New screens with no source → use `new-screen` instead
- Pure copy within the same app → just copy the file

## Hard rule

Do NOT write any code until alignment (step 1) is done and the user has approved the port spec. Skipping alignment on a port = wasted work, because web→mobile drops half the features and mobile→mobile shifts conventions.

## Required inputs

Before starting, you need:
1. **Source path** — file or folder of the screen being ported (e.g. `host/src/screens/Foo.tsx`)
2. **Target app** — where it's going (e.g. `app-host`)

If either is missing, ask the user for them first.

## Flow

### Step 1 — Align on the port (use `grill-with-docs` if installed)

Goal: nail down what carries over vs what drops, BEFORE touching code.

If `grill-with-docs` is installed, invoke it now with the source + target as context. Otherwise, run this inline:

Read the source file. Then ask the user, one question at a time:
1. **What's the user goal of this screen?** (re-confirm — may differ on mobile)
2. **What features MUST carry over?** (list explicitly)
3. **What features DROP for the target platform?** (mobile usually drops hover, multi-column, dense tables, keyboard shortcuts)
4. **What ADAPTS?** (e.g. table → cards, modal → bottom sheet, hover → long-press)
5. **What's the data source?** (same API? different endpoint? new endpoint needed?)
6. **What's the navigation context?** (where does the user arrive from / leave to in the target app)
7. **What's explicitly OUT of scope?** (kill scope creep early)

Stop and present a 5-line port spec. Wait for user approval before proceeding.

### Step 2 — Understand the SOURCE (use `zoom-out` if installed)

Goal: map the source screen's logic, state, and dependencies.

If `zoom-out` is installed, invoke it on the source path. Otherwise, read inline:
- The screen file itself
- Any hooks / services / store slices it depends on
- The data flow (where state lives, how API is called)
- Any non-obvious business rules embedded in the JSX

Produce a brief mental model: "source has X data, Y state transitions, Z user actions."

### Step 3 — Understand the TARGET app (use `zoom-out` if installed)

Goal: learn the target app's conventions before adapting.

If `zoom-out` is installed, invoke it on the target app's `src/`. Otherwise, read:
- Navigation structure
- 1-2 existing similar screens in the target (find via Glob)
- Store / service patterns
- Theme / design tokens
- Project-level `screen-builder.md` agent in `.claude/agents/` if it exists — READ IT, those are the target's rules

### Step 4 — Test the logic layer (use `tdd` if installed)

Goal: test the pieces that aren't pure JSX.

Write failing tests for (in this order):
1. **Service function** — if a new endpoint or new shape, test it with mocked axios
2. **Store action(s)** — state transitions for the ported screen
3. **Custom hooks** — anything non-trivial extracted from the JSX
4. **Validation / selectors** — pure functions

Do NOT snapshot-test the JSX. Do NOT test trivial rendering.

If `tdd` skill is installed, invoke it per piece. Otherwise red → green → refactor manually.

### Step 5 — Build the adapted screen

Delegate to the target project's worker agents. They already encode the target conventions — use them, don't reinvent.

**If a new API endpoint is needed in the target**, run agents in this order:
1. `api-integrator` — adds types, service function, store action
2. `screen-builder` — writes the adapted screen JSX + i18n + nav

**If the API already exists in the target**, run only:
1. `screen-builder`

Pass to each agent:
- The port spec from step 1
- The target conventions from step 3
- Pointer to source file for reference (not for copy-paste)

```
Agent(subagent_type="api-integrator", prompt="<endpoint spec if new>")
Agent(subagent_type="screen-builder", prompt="<spec + target conventions + source pointer>")
```

The agents should ADAPT the source to target conventions, not copy verbatim.

### Step 6 — Review

Delegate to the target project's `code-reviewer` agent. Pay extra attention to:
- Did we drift back into source conventions instead of target?
- Did we accidentally bring web-only patterns into mobile (or vice versa)?
- Are the logic-layer tests actually covering the ported behavior?

Fix any flagged issues before declaring done.

## What this skill does NOT do

- It does not modify the source — the source stays untouched
- It does not run the app or test on a device — remind the user
- It does not commit, push, or open PRs

## Output

When done, give the user a 3-line summary:
- What was ported (source → target paths)
- What was dropped / adapted (vs verbatim)
- What's left (manual device test, commit, PR)
