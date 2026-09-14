# AI Engineering Governance & TDD Rules

## Pre-Action Workflow (Mandatory Planning)
1. **Plan First:** Analyze requirements and write a step-by-step plan.
   - Mention affected files.
   - Define test strategy (how to achieve 100% coverage safely).
   - Justify any anticipated use of `# pragma: no cover`.
2. **User Confirmation:** State **[PLANNING]** and WAIT for user approval.

## TDD Workflow (Mandatory Loop)
State your current phase explicitly: **[RED] / [GREEN] / [REFACTOR] / [COMMIT]**.

### Step 1 — RED: Write a failing test first
- Write the test BEFORE modifying implementation.
- Run `make test`. Confirm it fails for the *expected reason*.

### Step 2 — GREEN: Write the minimum code to pass
- Write minimal code to pass the test (YAGNI).
- Re-run `make test`.
- *Rule:* If coverage drops, revert uncommitted changes, rethink, and try again.

### Step 3 — REFACTOR: Clean up
- Improve naming, extract methods. Re-run `make test`, `make lint`, `make typecheck`.

### Step 4 — COMMIT (Mandatory Final Step)
- Once tests pass and linters are clean, run `git add <specific_files>`.
- Create a Conventional Commit (e.g., `feat: ...`, `fix: ...`).
- Run `git commit -m "<message>"`. Never report a task as "done" without committing.
