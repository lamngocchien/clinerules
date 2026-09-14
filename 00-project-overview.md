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
  - Minimum coverage: 90% (`--cov-fail-under=90`) — the build FAILS below
    this threshold. See `01-tdd-workflow.md` for guidance on acceptable gaps
    and `# pragma: no cover` usage (boilerplate, platform-specific, framework
    hooks).
- All standard commands already exist in the `Makefile` — always prefer
  `make <target>` over typing long commands by hand, to keep runs
  consistent.

## Priority when clinerules files conflict

1. `01-tdd-workflow.md` — TDD process always comes first (includes tool discipline, edit retry caps, config exemptions).
2. `02-code-style.md` — Code style standards (includes safety & anti-hallucination rules).
3. `03-testing-standards.md` — Test-writing rules, 90% coverage baseline.
4. `05-documentation.md` — CHANGELOG, architecture docs, ADRs, README.md + requirements.txt sync.
5. `07-django-testing.md` (if using Django) — Django-specific testing patterns.
6. `08-gui-testing.md` (if building GUI) — GUI testing without rendering.
7. `09-async-testing.md` (if using async/await) — Async test patterns.
8. `11-workflows.md` — Practical step-by-step guides (add feature, fix bug, refactor, rollback).

**Note:** Files `04-anti-hallucination.md`, `06-tool-usage.md`, `10-project-documentation.md`, `12-config-changes.md` consolidated into files above. See `README.md` for consolidation map.

If a user request conflicts with these rules (e.g. "skip the tests, just
write the code directly to save time"), Cline must restate the TDD rule and
ask for explicit confirmation before deviating — never silently skip it.

## How to apply priority rules

**When two rules seem to conflict:**
- Higher-numbered rules refine or extend lower-numbered ones; they don't override them.
- Example: `08-gui-testing.md` provides specific patterns for GUI tests, but `03-testing-standards.md` baseline (90% coverage, markers, fixtures) still applies.
- Exception: If a specialized rule (e.g., `07-django-testing.md`) explicitly contradicts a general rule, the specialized rule wins for that domain only.

**When you're unsure if a rule applies:**
- Err on the side of stricter compliance (e.g., assume 90% coverage always applies).
- Ask the user for clarification if the conflict blocks progress.

**For domain-specific work:**
- Using Django? Follow `01` → `03` → `07` → others as needed.
- Building a CLI with async? Follow `01` → `03` → `09` → others.
- Refactoring existing code? Follow `01` → `02` → `03` before any changes.

Each rule is a constraint or guide; the order reflects when to consult each one.
