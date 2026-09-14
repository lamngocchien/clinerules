# AI Engineering Governance & TDD Rules

## Pre-Action Workflow (Mandatory Planning)
Before initiating any implementation, Cline must follow these steps:
1. **Plan First:** Analyze the requirements and write down a concise, step-by-step implementation plan covering:
   - Affected files (to be created or modified).
   - Test strategy (where the test will live, what it will assert, and how to maintain 100% branch/line coverage).
   - Potential edge cases or risks.
2. **User Confirmation & State:** Present the plan clearly to the user, state **[PLANNING]**, and **wait for explicit confirmation**. Never write implementation or test code before the plan is approved.

---

## TDD Workflow (Mandatory)
For **every** approved feature, function, class, or bugfix, Cline must follow the **Red → Green → Refactor → Commit** loop in this exact order:

### Step 1 — RED: Write a failing test first
- Write a test in `tests/unit/` (or `tests/integration/` if real I/O is needed) that describes the desired behavior, BEFORE writing any implementation code.
- Run the specific test path (e.g., `pytest <path>`) and confirm the test **fails for the expected reason** (e.g., `ImportError`, `AttributeError`, a wrong assertion) — not because of a syntax error in the test itself.
- Never write implementation code before at least one test is failing.

### Step 2 — GREEN: Write the minimum code to pass
- Write the **minimum** amount of code needed to make the Step 1 test pass. Do not add extra features, optimizations, or edge-case handling ("YAGNI").
- Re-run `make test` (or the test suite) and confirm the entire suite passes.
- If `make test` fails after GREEN (coverage drops below 100% or another test breaks), immediately revert uncommitted changes and diagnose the root cause before retrying. Never commit broken code or skip a failing test to "move forward."

### Step 3 — REFACTOR: Clean up while staying green
- Improve naming, structure, remove duplication, extract functions — but do NOT change externally observable behavior.
- After each small change, re-run `make test` to confirm it's still green.
- Run `make check` (combines `lint`, `typecheck`, and `test`) before considering the task done.

### Step 4 — COMMIT: Automatic Git Commit
- After successfully passing all tests, linters, type checks, and confirming 100% coverage:
  - Run `git add` specifically for the modified or created files related to the task (avoid `git add .`).
  - Automatically compose a clean, conventional commit message (e.g., `feat:`, `fix:`, `refactor:`) based strictly on the work done.
  - Execute `git commit -m "<message>"` and report the commit hash and message to the user.

---

## Tool Discipline & Avoiding Edit Loops

### Never retry a failed edit identically
If a file edit (diff/patch-based `old_str` match, or any similar find-and-replace edit) fails, do NOT resend the exact same call. Instead:

1. Re-read the file's current full content first — never assume it still matches what you last saw, especially if you or another step already edited it earlier in this task.
2. Diagnose why the match failed before retrying: whitespace/indentation mismatch, the target text appearing more than once, or the surrounding code having changed.
3. Retry with a corrected match based on what you just read — widen the matched context if it wasn't unique, or fix whitespace if that was the issue.

### Cap retries — escalate instead of looping
- Allow at most **2 attempts** at fixing the same failed edit. If it still fails on the second attempt, stop and explain to the user exactly what's blocking it (e.g. "the text I'm trying to match appears twice in the file" or "the file's indentation uses tabs, not spaces, which is throwing off my match") — do not keep retrying with no new information.
- Never issue more than 2 consecutive identical tool calls of any kind. If a second identical call would be made, stop and ask the user for guidance instead.

### Prefer full-file rewrite when uncertain
- For files under ~150 lines with simple structure (single function/class), if a targeted patch has failed once, prefer rewriting the entire file with the corrected content over attempting another partial diff — it removes the ambiguity that caused the failure.
- For larger or complex files (multiple classes, nested logic), narrow the edit to a smaller, more specific, and more clearly unique chunk of surrounding text rather than retrying the same large match.
- Always re-read the file before deciding whether to retry or rewrite — do not guess based on memory of earlier state.

### After any edit, verify before moving on
- After an edit succeeds, briefly re-read the changed section to confirm it matches intent before running tests — do not chain multiple edits to the same file without checking each one landed correctly.

### Commit message format (conventional commits)
- Format: `<type>(<scope>): <subject>` with optional body and footer.
- **Type:** `feat` (new feature), `fix` (bug fix), `refactor` (code restructure without behavior change), `test` (test-only changes), `docs` (documentation only), `chore` (tooling, deps, config).
- **Scope** (optional): affected module name (e.g., `auth`, `user-service`). Omit if change affects entire project.
- **Subject:** lowercase, imperative mood ("add feature" not "added feature"), no period, under 50 characters.
- **Body** (optional): detailed explanation of why the change was made, not what changed. Wrap at 72 characters.
- **Example:** `feat(auth): add JWT token refresh endpoint` or `fix: resolve race condition in cache invalidation`

### Pre-commit hooks

- Project includes pre-commit hooks (configured in `.pre-commit-config.yaml` if present). Hooks run `ruff`, `black`, and `mypy --strict` before each commit.
- Install hooks: `pre-commit install` (one-time setup).
- Run manually: `pre-commit run --all-files` to validate all files without committing.
- Hooks auto-fix code issues (`ruff --fix` may change logic, not just formatting). After hooks run:
  1. Review all changes (`git diff`) before staging — `ruff --fix` can reorder imports, remove unused code, or restructure conditions.
  2. Verify changes align with intent; revert with `git checkout` if not.
  3. Re-run tests (`make test`) after accepting fixes.
  4. Stage and commit only after confirming tests pass.

---

## Additional Rules
- **Explicit State Reporting:** Whenever a response includes a code change or status update, Cline must explicitly state its current step (**[PLANNING] / [RED] / [GREEN] / [REFACTOR] / [COMMIT]**) at the top of its explanation.
- **Atomic Tasks:** Do not bundle multiple unrelated features into a single Red-Green-Refactor cycle.
- **Bugfixes:** For bugfixes, the first step after plan approval is still to write a regression test that reproduces the bug.
- **Coverage (90% Standard):** Coverage baseline is **90%** (configured as `--cov-fail-under=90` in `pyproject.toml`). Every new feature must include tests covering the primary paths and common edge cases. Use `# pragma: no cover` judiciously for genuinely unreachable code.
  - Use `# pragma: no cover` **only** for code that is genuinely unreachable at runtime: exception handlers for system-level errors (e.g., `except OSError: # pragma: no cover`), platform-specific code paths (e.g., `if sys.platform == "win32": # pragma: no cover`), boilerplate/framework setup (e.g., `__init__.py` with only imports), or defensive assertions that guard against logic errors in calling code.
  - Never use `# pragma: no cover` to hide untested business logic or edge cases — write a test instead.
  - Document why each `# pragma: no cover` exists in a comment immediately above or on the same line. Do not use it without explanation.
  - **Acceptable gaps (<10%):** Boilerplate (fixture setup, class initialization), integration-test-only paths, platform-specific code, framework hooks (Django admin, signal handlers for UI-only events).
- **Test Integrity:** Never delete or weaken a failing test just to make it "pass" — fix the code or fix the test to match the actual requirement, and clearly explain why if the test genuinely was wrong.
- **Exemption:** Pure documentation updates, configuration files without business logic, or formatting changes are exempt from the TDD loop (but still require a brief plan).

## Configuration File Changes & TDD Exemption

This section defines when and how to handle edits to `pyproject.toml`, `pytest.ini`, `setup.cfg`, and other project configuration files.

### Classification: Config vs Code

**Config files (TDD exemption applies):**
- `pyproject.toml` — dependency versions, tool settings, metadata.
- `pytest.ini` / `setup.cfg` — test runner, linter, type checker configuration.
- `.pre-commit-config.yaml` — hook pipeline and versions.
- `Makefile` — build targets and commands.

**Not config (full TDD applies):**
- Source code in `src/app/`.
- Test code in `tests/`.
- `CHANGELOG.md`, `README.md`, ADRs — documentation updates (exempt but require plan).

### When config changes trigger tests

| Change | Requires tests? | Reason |
|--------|---|---|
| Add dependency to `pyproject.toml` | Yes | New package must be importable; may affect existing code. |
| Bump dependency version | No | Assumes version is compatible; if incompatible, discovered by `make check`. |
| Add `pytest` marker or plugin | No | Tool configuration; validated by running `make test`. |
| Add new `[tool.ruff]` rule | No | Linter config; validated by `make lint`. |
| Change `mypy` strictness settings | No | Type checker config; validated by `make typecheck`. |
| Increase coverage threshold (e.g., 90% → 95%) | No | Config change; build fails if coverage drops, forcing code fixes. |
| Add build phase or script | Yes if it touches source code | If script generates code, modifies files, or runs custom tests, treat as code change. |

### Workflow: Config-only changes

**Exempt from TDD loop**, but still require a plan:

1. **State the change:** "Bump pytest from 7.0 to 8.0" or "add `[tool.black]` line-length=100".
2. **Reason:** Why the change (compatibility, new feature, stricter linting).
3. **Verify:** Run `make check` to ensure no breakage.
4. **Commit:** Use `chore:` prefix if no code changes.
   - Example: `chore(deps): bump pytest to 8.0`
   - Example: `chore(config): add ruff isort integration to pyproject.toml`

### Workflow: Config + code changes

If a config change *requires* code changes to pass:

1. Start with the code change (follow full TDD: RED → GREEN → REFACTOR → COMMIT).
2. Update config as part of REFACTOR (e.g., new dependency, tool setting).
3. Run `make check` before committing.
4. Commit message includes both: `feat(auth): add JWT; config: bump cryptography to 42.0`.

### Anti-patterns for config changes

- ❌ Bump dependency version without checking `make check` first — hidden incompatibility.
- ❌ Increase coverage threshold without fixing code to meet it — build will fail.
- ❌ Add new tool to `pyproject.toml` but skip running `make check` — may not be installed.
- ❌ Commit config changes without stating reason — unclear intent for future readers.

### Pre-commit hook interaction with config

Config files (especially `pyproject.toml`) are checked by pre-commit hooks:
- `ruff` validates TOML syntax.
- `black` may reformat multiline strings in config.

After editing config:
1. Run `pre-commit run <file>` to validate.
2. Review changes (`git diff`).
3. Re-run `make check` to confirm no side effects.
4. Commit.