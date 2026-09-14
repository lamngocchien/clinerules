# TDD Workflow Templates

Practical step-by-step guides for common development tasks, reinforcing the Red → Green → Refactor → Commit cycle from `01-tdd-workflow.md`.

**Workflows (inline in this file):**
- Add a New Feature
- Fix a Bug
- Refactor Without Changing Behavior
- Add a Test for Existing Code

**Standalone workflow guides (in `workflows/` directory):**
- `workflows/new-feature.md` — Cline `/new-feature` command
- `workflows/fix-bug.md` — Cline `/fix-bug` command
- `workflows/rollback.md` — Emergency rollback/revert procedures

## Template: Add a New Feature

**Scenario:** Add a new user-facing function or feature (e.g., "calculate discount based on order total").

### [PLANNING]

1. **Understand requirement:** What does the feature do? What are edge cases?
2. **Identify affected files:**
   - New module in `src/app/` or change existing module?
   - Test file in `tests/unit/` or `tests/integration/`?
3. **Plan test strategy:**
   - Happy path (normal input, expected output).
   - Edge cases (empty, negative, boundary values).
   - Error cases (invalid input raises correct exception).
4. **Confirm with user:** Present the plan. Get approval before coding.

### [RED] Write failing test first

- Create test in `tests/unit/test_<module>.py` (or integration if needed).
- Write test that describes the desired behavior.
- Run `pytest tests/unit/test_<module>.py` — confirm **test fails** for the right reason (e.g., `ImportError`, function doesn't exist, wrong return value).
- Never skip this step; red tests validate your test itself.

### [GREEN] Write minimum code

- Add the function/method to `src/app/<module>.py`.
- Write the **minimal** code to pass the test. No optimizations, no extra features.
- Run `make test` — confirm entire suite passes, 100% coverage maintained.
- If coverage drops, add tests for new branches.

### [REFACTOR] Improve while staying green

- Rename variables/functions for clarity.
- Extract helpers to reduce duplication.
- Add docstring (Google-style, with `Args`, `Returns`, `Raises`).
- Add type hints (run `make typecheck` — must pass `mypy --strict`).
- Run `make test` after each small change.
- Run `make lint` (ruff) — fix any style violations.

### [COMMIT]

- Stage files: `git add src/app/<module>.py tests/unit/test_<module>.py`
- Commit: `git commit -m "feat(<module>): add <feature name>"`
- Example: `git commit -m "feat(orders): add calculate_discount function"`

### After commit: Update docs

- Update `src/README.md` — add feature description under "Features" section.
- If new package imported, add to `src/requirements.txt` with pinned version.
- Update `CHANGELOG.md` — add entry under `## [Unreleased] - Added`.
- Commit docs: `git add src/README.md CHANGELOG.md` → `git commit -m "docs: update README and CHANGELOG for discount feature"`

---

## Template: Fix a Bug

**Scenario:** A user reports incorrect behavior (e.g., "discount calculation returns negative values when it shouldn't").

### [PLANNING]

1. **Reproduce the bug:**
   - Create a minimal test case that demonstrates the bug.
   - Run it — confirm it fails in the way the user described.
2. **Identify root cause:**
   - Read the implementation. Is it a logic error? Boundary condition?
3. **Plan the fix:**
   - What code change will fix the root cause?
   - Will the fix break any existing tests?

### [RED] Write regression test

- Create test in `tests/unit/test_<module>.py` that reproduces the bug.
- Test should **fail** before the fix, **pass** after.
- Example: `test_discount_never_negative_for_high_order_value()`
- Run test — confirm it fails with the bug behavior.

### [GREEN] Fix the code

- Modify the implementation in `src/app/<module>.py` to fix the bug.
- **Minimal fix only** — don't refactor unrelated code.
- Run `make test` — confirm all tests pass, including the new regression test.

### [REFACTOR]

- No refactoring needed if the fix is already minimal.
- If the fix exposed other issues (e.g., duplicated boundary checks), extract helpers.
- Run `make lint`, `make typecheck`.

### [COMMIT]

- Commit: `git commit -m "fix(<module>): prevent negative discounts"`

### After commit: Update docs

- Update `CHANGELOG.md` — add entry under `## [Unreleased] - Fixed`.
- If behavior changed in a user-facing way, update `src/README.md`.

---

## Template: Refactor Without Changing Behavior

**Scenario:** Code is hard to read, has duplication, or violates style rules. No new features, no bug fixes — only structure/clarity.

### [PLANNING]

1. **Scope the refactor:**
   - Which files/functions need improvement?
   - What's the target? (e.g., extract helper, rename variable, remove duplication)
2. **Ensure full test coverage:**
   - Run `make test-cov` — confirm the code being refactored has 100% coverage.
   - If not, add tests **before** refactoring. You need a safety net.

### [GREEN] Refactor incrementally

- Make one small change (rename, extract, simplify).
- Run `make test` after each change — tests must stay green.
- If a test breaks, revert and reconsider (refactor revealed a misunderstanding).
- Repeat until refactor is complete.

### [REFACTOR] Already done — no second pass needed

- The refactoring IS the refactor step.

### [COMMIT]

- Commit: `git commit -m "refactor(<module>): extract discount calculation logic"`

### After commit: Update docs

- Update `CHANGELOG.md` — add entry under `## [Unreleased] - Changed` (if user-facing) or skip (if internal-only).
- No need to update `README.md` unless the user-facing interface changed.

---

## Template: Add a Test for Existing Code

**Scenario:** You find untested code (coverage gap) or want to ensure edge cases are covered.

### [PLANNING]

1. **Identify the gap:**
   - Run `make test-cov` — find lines/branches with no tests.
   - Read the code — what could go wrong?
2. **Plan test cases:**
   - Happy path (if not already tested).
   - Boundary conditions (min/max, empty, None).
   - Error conditions (invalid input, exceptions).

### [RED] Write test that currently fails or is skipped

- Create test for the gap.
- If the test should pass (the code already handles it correctly), good — you've validated the code.
- If the test should fail (you found a bug), mark it `@pytest.mark.skip("Bug: <description>")` until you're ready to fix it.
- Or, if fixing now, follow the **Fix a Bug** template.

### [GREEN] Code already exists

- If the code is correct, test will pass immediately.
- If the code is wrong, fix it (see **Fix a Bug** template).

### [COMMIT]

- Commit: `git commit -m "test: add edge case coverage for discount_calculation"`

---

## Template: Rollback to Previous Commit or Tag

Emergency revert to stable state. See `workflows/rollback.md` for full step-by-step guide including:
- Identify rollback target (commit hash, tag, deployment snapshot)
- Choose method (revert, reset, automated GitHub Actions)
- Validate rollback with `make test`
- Document reason and fix forward

Quick reference: `git revert <hash>` (safe, audit trail) or `git reset --hard <hash>` (destructive, clean history).

---

## Checklist: Before Marking Task "Done"

- [ ] `make test` passes (100% coverage).
- [ ] `make lint` passes (ruff + black).
- [ ] `make typecheck` passes (`mypy --strict`).
- [ ] New/modified docs updated (`README.md`, `CHANGELOG.md`, ADRs if needed).
- [ ] Commit message follows conventional format (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`).
- [ ] No uncommitted changes.
- [ ] Branch is ready for PR (or merge to main if solo).

---

## Anti-Patterns

- ❌ **Skipping RED:** "I'll write the test after the code." → You lose the safety net.
- ❌ **Over-refactoring in GREEN:** "I'll clean this up while passing." → Tests may break; stick to minimal.
- ❌ **Ignoring coverage drops:** "It's only 98%, close enough." → Build fails; fix it now.
- ❌ **Committing without docs:** "I'll update README later." → Later never comes; docs decay.
- ❌ **Multiple features in one commit:** "I'll do auth + logging together." → Harder to revert, harder to review. Use separate cycles.

---

## When Stuck

| Problem | Solution |
|---------|----------|
| Test won't fail in RED | Test may be wrong. Re-read requirement. Does the test actually test what you think? |
| Code won't pass in GREEN | Logic error or test too strict. Read error message. Adjust code or test (if test was wrong). |
| Coverage stays at 100% but feels incomplete | Branch coverage gap. Use `make test-cov --cov-report=term-missing` to find uncovered lines. |
| Refactor breaks multiple tests | Refactor was too broad. Revert. Do it in smaller steps. One logical change per cycle. |
| Forgot to update docs | Go back and update. Commit docs separately (`docs:` prefix). This is part of "done." |

