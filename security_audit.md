# Security Audit Workflow

This workflow is invoked as `/security-audit`. Use it right after making significant code changes, before creating a release, or periodically, to perform static application security testing (SAST) using **Bandit** to identify common security vulnerabilities, hardcoded secrets, and unsafe coding patterns in the Python codebase. Follow these steps **in order**:

## Step 1 — Check Bandit availability & environment
- Verify that `bandit` is installed and accessible in the current Python environment:
  ```bash
  bandit --version
  ```
- If bandit is missing from the environment, state so explicitly and ensure it is added to development dependencies in `pyproject.toml` before proceeding.

## Step 2 — Run the security scan
- Execute the bandit scan across the project source tree (`src/`) using filtered confidence and severity parameters (`-ll -ii` to reduce false positives):
  ```bash
  bandit -r src/ -ll -ii
  ```
- If a project `Makefile` security target exists (e.g., `make security`), prefer using it for consistency.

## Step 3 — Triage and remediate findings
- Review every reported issue from the scan output:
  - **True Positive:** A real security risk (e.g., hardcoded credentials, command injection, weak cryptography). Fix it by following the standard TDD / bug-fix workflow (`01-tdd-workflow.md`).
  - **False Positive / Intentional Trade-off:** Annotate the line with `# nosec BXXX` and a mandatory inline comment explaining why it is safe in context.
- Re-run the scan to confirm all true positives are fully resolved.

## Step 4 — Verify quality gates and document
- Run `make check` to ensure code style, type checking (`mypy --strict`), and test coverage (90%+) remain fully intact with no regressions.
- Update project documentation per `.clinerules/05-documentation.md`:
  - Add a bullet to `CHANGELOG.md` under `## [Unreleased] > Fixed` if any vulnerabilities or security hardening were addressed.
  - If the audit establishes a new architectural security constraint, add an ADR under `docs/decisions/`.
