---
name: build-feature
description: This skill should be used when the user asks to "build a feature", "add a feature", "start a new feature", "implement X", "code the search", "restructure into layers", or otherwise begins non-trivial coding work in this Go backend. It enforces the project workflow — discuss & confirm first, TDD, coding-standards, layered handler/service/repository structure, code-review, and ask before commit.
---

# Build Feature — project workflow

Follow this pipeline IN ORDER for every non-trivial feature or code change in this repo.
Do not skip steps. The source of truth for these rules is `CLAUDE.md` at the repo root;
if anything here conflicts with `CLAUDE.md`, `CLAUDE.md` wins.

## 1. Discuss & confirm first (no code yet)

- Restate the goal in plain words and surface ambiguity (scope, edge cases, files touched).
- Propose the folder structure and the plan.
- Use the `brainstorming` skill to explore intent and design.
- Use the `grill-with-docs` skill to pressure-test the plan against the existing code/domain.
- Wait for explicit user approval before writing or editing ANY code.

## 2. TDD

- Use the `tdd` skill.
- Write failing tests first (red), then the minimum code to pass (green), then refactor.
- Cover each layer: service orchestration, repository mapping, handler response shape.
- Use the AAA pattern (Arrange / Act / Assert); include error paths.

## 3. Implement to coding-standards

- Use the `coding-standards` skill.
- Typed structs + `json.Unmarshal` — never `gjson` string-path digging.
- Named constants for magic strings/numbers (query params, timeouts, env keys).
- Small single-purpose functions; early returns; avoid deep nesting.
- Layered structure: each feature in its own folder with `handler` → `service` → `repository`
  (e.g. `search/handler.go`, `search/service.go`, `search/repository.go`).
- Comments in Vietnamese; identifiers and commit messages in English.

## 4. Code-review

- Run the `code-review` skill on the change.
- Address findings before considering the work done.

## 5. Ask before commit

- Never run `git commit` without explicit user confirmation.

## Layered architecture reference

- **handler** — HTTP layer (gin): bind request, call service, return JSON. No business logic.
- **service** — orchestration & business rules (e.g. provider fallback order).
- **repository** — data sources behind a shared interface (external API, embedded data).
  Swapping a provider = change the repository wired into the service, nothing else.
