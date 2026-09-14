# Cline Rules: Project Governance & TDD Standards

Consolidated governance framework for AI-assisted development using Test-Driven Development (TDD).

## File Structure (9 files)

**Core TDD & Workflow:**
- `00-project-overview.md` — Project layout, tooling overview, rule priority order.
- `01-tdd-workflow.md` — TDD cycle (RED → GREEN → REFACTOR → COMMIT), tool discipline, config exemptions.
- `02-code-style.md` — PEP 8, naming, type hints, docstrings, safety & anti-hallucination rules.
- `03-testing-standards.md` — Test structure, markers, mocking, fixtures, coverage (90%+), edge cases.

**Documentation & Design:**
- `04-documentation.md` — CHANGELOG, architecture, ADRs, README.md + requirements.txt sync rules, deprecation patterns.

**Domain-Specific Testing:**
- `05-django-testing.md` — Django models, views, serializers, auth, database isolation, signals.
- `06-gui-testing.md` — Headless GUI logic testing, event injection, mocking, state isolation.
- `07-async-testing.md` — Async/await patterns, fixtures, mocking, concurrency, task cancellation.

**Workflows & Guides:**
- `08-workflows.md` — Step-by-step templates: add feature, fix bug, refactor, test gaps. Rollback guide reference.

## Consolidation Summary

**Merged into existing files (reduced from 12 → 9):**
- `04-anti-hallucination.md` → `02-code-style.md` (new "Safety & Anti-Hallucination" section)
- `06-tool-usage.md` → `01-tdd-workflow.md` (new "Tool Discipline & Avoiding Edit Loops" section)
- `10-project-documentation.md` → `04-documentation.md` (expanded "Project Documentation Standards" section)
- `12-config-changes.md` → `01-tdd-workflow.md` (new "Configuration File Changes & TDD Exemption" section)

**Preserved standalone (domain-specific extensions to `03-testing-standards.md`):**
- `05-django-testing.md`, `06-gui-testing.md`, `07-async-testing.md` remain separate for clarity.

## Rule Priority

1. `01-tdd-workflow.md` — TDD process always comes first.
2. `02-code-style.md` — Code style standards (includes anti-hallucination).
3. `03-testing-standards.md` — Test-writing rules, 90% coverage baseline.
4. `04-documentation.md` — CHANGELOG, architecture docs, README sync.
5. `05-django-testing.md` — Django-specific patterns (if using Django).
6. `06-gui-testing.md` — GUI testing without rendering (if building GUI).
7. `07-async-testing.md` — Async test patterns (if using async/await).
8. `08-workflows.md` — Practical step-by-step guides for common tasks.

**Higher-numbered rules refine lower ones; they don't override them.** Exception: specialized rules (e.g., `05-django-testing.md`) win for their domain only.

## When to Use Each File

| Task | Consult |
|------|---------|
| Starting a new feature or bug fix | `01-tdd-workflow.md` + `03-testing-standards.md` |
| Writing code (naming, types, imports) | `02-code-style.md` |
| Setting up tests (markers, fixtures, mocking) | `03-testing-standards.md` |
| Updating docs after a change | `05-documentation.md` |
| Testing Django models/views | `07-django-testing.md` + `03-testing-standards.md` |
| Testing GUI logic (no rendering) | `08-gui-testing.md` + `03-testing-standards.md` |
| Testing async functions | `09-async-testing.md` + `03-testing-standards.md` |
| Following a workflow template | `11-workflows.md` |
| Understanding project structure | `00-project-overview.md` |

## Key Principles

- **TDD is mandatory:** RED (failing test) → GREEN (min code) → REFACTOR (clean up) → COMMIT.
- **Coverage baseline: 90%** with all tests passing before commit.
- **Config exemption:** Changes to `pyproject.toml`, `pytest.ini`, `Makefile` exempt from TDD if logic-free; config + code changes follow full TDD for code part.
- **Documentation is done-done:** Update CHANGELOG, README.md, architecture docs as part of REFACTOR, not after.
- **No fabrication:** Never invent APIs, functions, or packages. State "unconfirmed" if unsure.
- **Edit discipline:** Re-read files before retry. Max 2 attempts per failed edit; escalate if blocked.

## Workflow & Commit Format

**Conventional commits:**
- `feat(scope): add feature`
- `fix(scope): resolve bug`
- `refactor(scope): restructure code`
- `test: add test coverage`
- `docs: update README/CHANGELOG`
- `chore(deps): bump dependency`

**Pre-commit hooks validate:** ruff (lint), black (format), mypy --strict (type check). Review changes before committing.

## See Also

- `.workflows/` subdirectory: Standalone guides for Cline `/new-feature`, `/fix-bug`, emergency rollback.
- `src/README.md` — User-facing project documentation (features, installation, usage).
- `src/requirements.txt` — Pinned runtime dependencies (must stay in sync with code).
- `CHANGELOG.md` — Project history (updated with every user-visible change).
- `docs/architecture.md` — Current system design (updated when structure changes).
- `docs/decisions/` — Architecture Decision Records (ADRs) for non-obvious choices.
