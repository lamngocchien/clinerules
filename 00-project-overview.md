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

If a user request conflicts with these rules (e.g. "skip the tests, just
write the code directly to save time"), Cline must restate the TDD rule and
ask for explicit confirmation before deviating — never silently skip it.
