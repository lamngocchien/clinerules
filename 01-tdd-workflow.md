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
- Run `make lint` and `make typecheck` before considering the task done.

### Step 4 — COMMIT: Automatic Git Commit
- After successfully passing all tests, linters, type checks, and confirming 100% coverage:
  - Run `git add` specifically for the modified or created files related to the task (avoid `git add .`).
  - Automatically compose a clean, conventional commit message (e.g., `feat:`, `fix:`, `refactor:`) based strictly on the work done.
  - Execute `git commit -m "<message>"` and report the commit hash and message to the user.

---

## Additional Rules
- **Explicit State Reporting:** Whenever a response includes a code change or status update, Cline must explicitly state its current step (**[PLANNING] / [RED] / [GREEN] / [REFACTOR] / [COMMIT]**) at the top of its explanation.
- **Atomic Tasks:** Do not bundle multiple unrelated features into a single Red-Green-Refactor cycle.
- **Bugfixes:** For bugfixes, the first step after plan approval is still to write a regression test that reproduces the bug.
- **Coverage (100% Strict):** Coverage must always stay strictly **100%** (configured as `--cov-fail-under=100` in `pyproject.toml`). Every single branch, line, and edge case introduced must have a corresponding test. Never lower the threshold to bypass the check.
  - Use `# pragma: no cover` **only** for code that is genuinely unreachable at runtime: exception handlers for system-level errors (e.g., `except OSError: # pragma: no cover`), platform-specific code paths (e.g., `if sys.platform == "win32": # pragma: no cover`), or defensive assertions that guard against logic errors in calling code.
  - Never use `# pragma: no cover` to hide untested business logic or edge cases — write a test instead.
  - Document why each `# pragma: no cover` exists in a comment immediately above or on the same line. Do not use it without explanation.
- **Test Integrity:** Never delete or weaken a failing test just to make it "pass" — fix the code or fix the test to match the actual requirement, and clearly explain why if the test genuinely was wrong.
- **Exemption:** Pure documentation updates, configuration files without business logic, or formatting changes are exempt from the TDD loop (but still require a brief plan).