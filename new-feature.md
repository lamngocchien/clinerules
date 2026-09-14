# New Feature / Change Request Workflow

This workflow is invoked as `/new-feature` in the Cline chat. The user will
describe the desired feature or change right after the command. Follow
these steps **in order**:

## Step 0 — Check existing design first
- Read `docs/architecture.md` and skim `docs/decisions/` for any ADR that
  touches the area being changed.
- If the request conflicts with an existing ADR's decision, say so
  explicitly and ask whether to supersede it before writing any code.

## Step 1 — RED: Write the test first
- Write a test in `tests/unit/` (or `tests/integration/`) describing the
  desired behavior, before any implementation.
- Run `make test` and confirm it fails for the expected reason.

## Step 2 — GREEN: Minimum implementation
- Write just enough code to make the test pass. No speculative extras.
- Re-run `make test` and confirm the full suite passes.

## Step 3 — REFACTOR
- Clean up naming/structure without changing behavior, staying green
  throughout.
- Run `make check` (lint + typecheck + test).

## Step 4 — Update project documentation (mandatory, not optional)
Per `.clinerules/05-documentation.md`:
- Add a bullet to `CHANGELOG.md` under `## [Unreleased]`.
- If module responsibilities or data flow changed, edit the relevant
  section of `docs/architecture.md` in place.
- If a non-trivial technical decision was made (library choice, trade-off,
  a constraint the user imposed), add a new ADR in `docs/decisions/`
  using `docs/decisions/0000-template.md`, with the next sequential number.

## Reporting back
After each step, state which step just completed (RED / GREEN / REFACTOR /
docs updated) and show the actual command output — never claim a step
passed without having run it in this session.
