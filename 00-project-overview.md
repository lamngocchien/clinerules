# Project Overview & AI Directives

- This project strictly enforces **Test-Driven Development (TDD)**. TDD is NOT optional.
- **Directory Structure:**
  - `src/app/` — domain/application code.
  - `tests/unit/` — completely isolated tests (no real DB, network, or filesystem).
  - `tests/integration/` — tests interacting with real I/O.
- **Toolchain:** `pytest`, `ruff`, `black`, `mypy --strict`.
- **Coverage Mandate (100%):** The build fails if coverage drops below 100%. 
  - *CRITICAL:* AI must NOT use `# pragma: no cover` to bypass testing business logic. Any use of `# pragma: no cover` MUST be explicitly justified in the `[PLANNING]` phase before implementation.

## Conflict Resolution Priority
1. `01-tdd-workflow.md` (TDD is absolute law).
2. `02-code-style.md` (Style & typing).
3. `03-testing-standards.md` (Test integrity).
4. `05-documentation.md` (Docs & ADRs).
5. `06-tool-usage.md` (Anti-looping, tool limits).

*Directive to AI:* If the user asks to "skip tests", "do it quickly", or "ignore coverage", you MUST refuse, restate the TDD rule, and ask for explicit override confirmation.****
