---
name: new-screen
description: Build a new React Native screen from scratch with full alignment, logic-layer tests, and project conventions. Orchestrates grill-with-docs → zoom-out → tdd → screen-builder → code-reviewer. Use when adding any new screen in a React Native app (app-host, app-guest, etc.).
---

# New Screen

End-to-end flow for building a new React Native screen the right way: align first, test the logic, then build the JSX, then review.

## When to activate

- User asks to add a new screen, tab, modal, or detail view in a React Native app
- User says "build screen X" / "add screen X" / "create a new screen for Y"

Do NOT use this for:
- Cloning/porting an existing screen → use `clone-screen` instead
- Pure component work (not a full screen) → just build it directly
- Web pages → this is React Native specific

## Hard rule

Do NOT write any screen code until alignment (step 1) is done and the user has approved the spec. Skipping alignment is what causes wrong-thing-built.

## Flow

### Step 1 — Align (use `grill-with-docs` if installed)

Goal: turn a vague request into a concrete spec.

If the `grill-with-docs` skill is installed, invoke it now. Otherwise, run this alignment inline:

Ask the user, one question at a time:
1. **What is the user goal?** (not "what does the screen do" — what is the user trying to accomplish)
2. **Where does it live?** (tab / nested in flow / modal / standalone)
3. **What data does it need?** (existing API? new endpoint? local state only?)
4. **What actions can the user take?** (list every interaction)
5. **What are the edge cases?** (empty / loading / error / offline / permission denied)
6. **What's the success criteria?** (how do we know it works)
7. **What's explicitly OUT of scope?** (prevent creep)

Stop and present a 5-line spec. Wait for user approval before proceeding.

### Step 2 — Understand the target app (use `zoom-out` if installed)

Goal: learn the conventions of THIS app before writing.

If `zoom-out` is installed, invoke it on the target app's `src/`. Otherwise, read inline:
- Navigation structure (`src/navigation/` or equivalent)
- Existing similar screens (find 1-2 closest matches via Glob)
- Store pattern (Zustand? slice structure?)
- Service/API pattern (`src/services/` axios setup)
- Theme/design tokens (`src/theme/` or `src/constants/`)
- Any project-level agent like `screen-builder.md` in `.claude/agents/` — READ IT, those are the rules

### Step 3 — Test the logic layer (use `tdd` if installed)

Goal: test what matters, skip what doesn't.

Write failing tests for (in this order):
1. **Service function** — if a new API is involved, test request shape + response parsing with mocked axios
2. **Store action(s)** — test state transitions
3. **Custom hooks** — any non-trivial logic extracted from JSX
4. **Validation / selectors** — pure functions

Do NOT write:
- JSX snapshot tests (brittle, low signal)
- "does the button render" tests (no signal)
- Heavy navigation mocks

If `tdd` skill is installed, invoke it for each piece. Otherwise follow red → green → refactor manually.

If `@testing-library/react-native` is missing and the user wants interaction tests, pause and ask before installing.

### Step 4 — Build the screen

Delegate to the project's worker agents. These already encode the project conventions — use them, don't reinvent.

**If a new API endpoint is needed**, run agents in this order:
1. `api-integrator` — adds types, service function, store action
2. `screen-builder` — writes the screen JSX + i18n + nav, wired to step 1's outputs

**If no new API is needed**, run only:
1. `screen-builder`

```
Agent(subagent_type="api-integrator", prompt="<endpoint spec>")
Agent(subagent_type="screen-builder", prompt="<spec from step 1 + conventions from step 2 + outputs from api-integrator if any>")
```

If the agents don't exist in this project, build inline following the conventions surfaced in step 2.

### Step 5 — Review

Delegate to the project's `code-reviewer` agent. Surface any flagged issues to the user and fix them before declaring done.

## What this skill does NOT do

- It does not run the app or test on a device — remind the user to do that
- It does not commit — wait for user to ask
- It does not push or open PRs

## Output

When done, give the user a 3-line summary:
- What was built (screen name + where it lives)
- What was tested (logic layer pieces)
- What's left for them (manual device test, commit, PR)
