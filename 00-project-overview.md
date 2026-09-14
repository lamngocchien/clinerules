# Project Overview

- This is a Python project that treats **Test-Driven Development (TDD)** as
  the mandatory development process, not an optional practice.
- The directory layout follows the standard `src/` layout:
  - `src/app/` — application source code (rename the `app` package to match
    your real domain as soon as the project starts for real; update
    `pyproject.toml` accordingly).
  - `tests/unit/` — unit tests, no real I/O (filesystem, network, DB,
    subprocess).
  - `tests/integration/` — tests that touch real I/O or exercise several
    modules together.
- Standard project tooling (already configured in `pyproject.toml`):
  - Test runner: `pytest` (+ `pytest-cov`, `pytest-mock`).
  - Lint: `ruff`. Formatting: `black`. Type checking: `mypy --strict`.
  - Minimum coverage: 100% (`--cov-fail-under=100`) — the build FAILS below
    this threshold. Use `# pragma: no cover` only for unreachable system
    defenses (exception handlers for truly fatal errors, OS-specific code
    paths not testable on current platform).
- All standard commands already exist in the `Makefile` — always prefer
  `make <target>` over typing long commands by hand, to keep runs
  consistent.

## Priority when clinerules files conflict

1. `01-tdd-workflow.md` — the TDD process is always followed first.
2. `02-code-style.md` — code style standards.
3. `03-testing-standards.md` — detailed test-writing rules.
4. `04-anti-hallucination.md` — safety, avoiding fabricated APIs/libraries.
5. `05-documentation.md` — keeping CHANGELOG, architecture docs, and ADRs
   in sync with every change.
6. `06-tool-usage.md` — avoiding edit-retry loops; re-read before retrying
   a failed edit, cap retries, escalate instead of looping.
7. `07-django-testing.md` (if using Django) — Django-specific testing patterns.
8. `08-gui-testing.md` (if building GUI) — GUI testing without rendering.
9. `09-async-testing.md` (if using async/await) — async test patterns.
10. `10-project-documentation.md` — README and requirements.txt sync rules.
11. `11-workflows.md` — practical step-by-step guides for common tasks.

If a user request conflicts with these rules (e.g. "skip the tests, just
write the code directly to save time"), Cline must restate the TDD rule and
ask for explicit confirmation before deviating — never silently skip it.

## How to apply priority rules

**When two rules seem to conflict:**
- Higher-numbered rules refine or extend lower-numbered ones; they don't override them.
- Example: `08-gui-testing.md` provides specific patterns for GUI tests, but `03-testing-standards.md` baseline (100% coverage, markers, fixtures) still applies.
- Exception: If a specialized rule (e.g., `07-django-testing.md`) explicitly contradicts a general rule, the specialized rule wins for that domain only.

**When you're unsure if a rule applies:**
- Err on the side of stricter compliance (e.g., assume 100% coverage always applies).
- Ask the user for clarification if the conflict blocks progress.

**For domain-specific work:**
- Using Django? Follow `01` → `03` → `07` → others as needed.
- Building a CLI with async? Follow `01` → `03` → `09` → others.
- Refactoring existing code? Follow `01` → `06` → `02` before any changes.

Each rule is a constraint or guide; the order reflects when to consult each one.
