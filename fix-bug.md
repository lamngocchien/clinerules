# Fix Bug Workflow

This workflow is invoked as `/fix-bug` in the Cline chat. The user will
provide a bug description right after the command (symptom, input that
triggers it, expected vs actual behavior). Follow these steps **in order**,
without skipping or reordering any of them:

## Step 1 — RED: Write a regression test first
- Write a new test that reproduces the exact bug described, in the
  appropriate `tests/unit/` or `tests/integration/` file.
- Run `make test` and show the user that this specific test **fails**, and
  confirm the failure reason matches the reported bug (not an unrelated
  error).
- Do not touch implementation code yet.

## Step 2 — GREEN: Fix with the minimum necessary change
- Modify only what's needed to make the regression test pass.
- Do not refactor unrelated code or add unrequested improvements at this
  stage.
- Re-run `make test` and confirm the new test now passes.

## Step 3 — Run the full suite
- Run `make test` for the entire project (not just the new test) to confirm
  the fix does not break any existing behavior.
- If anything else breaks, treat it as a new RED step for that regression
  before continuing.

## Step 4 — REFACTOR (only if needed)
- If the fix left the code messy, clean it up now while keeping all tests
  green.
- Run `make check` (lint + typecheck + test) as the final gate before
  reporting the task done.

## Reporting back
After each step, explicitly tell the user which step just completed
(RED / GREEN / full-suite / REFACTOR) and the actual command output —
never claim a step passed without having run it in this session.
