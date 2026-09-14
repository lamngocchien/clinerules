# Debug Workflow

This workflow is invoked as `/debug`. The user will paste a raw error —
a Python traceback, a pytest failure output, a console error, or similar —
right after the command, often with little or no extra description.

## Step 1 — Parse the error before doing anything else
- Extract: exception type, error message, the file path(s) and line
  number(s) implicated, and which test or entry point triggered it (if the
  traceback shows a call chain).
- If the pasted text is too vague to locate a specific file/line (e.g. "it
  crashes sometimes" with no actual traceback), stop and ask the user to
  paste the full traceback or console output — do not guess the cause from
  an incomplete error.

## Step 2 — Read the real code, not memory of it
- Open every file referenced in the traceback and read the exact lines
  around the failure point in this session. Do not assume the file matches
  what it looked like earlier in the conversation.

## Step 3 — State a hypothesis before writing any code
- In one or two sentences, say what you believe is causing the error and
  why, based on what you just read. If the traceback's failing line is
  likely just where a bad value *surfaces* (not where it originates), trace
  back to the actual source before proceeding.

## Step 4 — RED: write a regression test
- Write a test with a minimal input that reproduces this exact error (same
  exception type / same wrong output).
- Run `make test` and confirm it fails with the same error signature the
  user pasted — if it fails differently, your reproduction is wrong; revise
  it before continuing.

## Step 5 — GREEN: minimal fix
- Fix the actual root cause identified in Step 3, not just the symptom at
  the reported line number.
- Re-run `make test` and confirm the regression test now passes.

## Step 6 — Full suite, then refactor
- Run `make test` for the whole project to check for regressions elsewhere.
- Refactor only if needed, staying green throughout, then run `make check`.

## Step 7 — Update documentation
Per `.clinerules/05-documentation.md`:
- Add a bullet to `CHANGELOG.md` under `## [Unreleased] > Fixed`.
- If the root cause exposes a design flaw significant enough to need a
  decision (not just a typo/off-by-one), add an ADR in `docs/decisions/`.

## Reporting back
- State which step just completed (parsed / hypothesis / RED / GREEN /
  full-suite / REFACTOR / docs updated) with actual command output.
- If an edit needs correcting, follow `.clinerules/06-tool-usage.md`:
  re-read the file before retrying, cap at 2 attempts, escalate instead of
  looping.
- If the same error is pasted again later in a different conversation, do
  not assume it's the same bug already fixed — repeat Steps 1–3 to confirm
  before reusing a prior hypothesis.
