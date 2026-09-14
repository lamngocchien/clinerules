# Setup Virtual Environment Workflow

This workflow is invoked as `/setup-venv`. It sets up a working Python
environment for this project from a fresh checkout. It is an environment
task, not a code change — it does not follow the Red-Green-Refactor loop
and does not require a CHANGELOG/ADR update unless it also modifies
`pyproject.toml` dependencies (see the note in Step 5).

## Step 1 — Check the Python version
- Run a command to check the available Python version (e.g. `python3
  --version`).
- This project requires **Python >= 3.11** (see `requires-python` in
  `pyproject.toml`). If the available version is older, or `python3` is
  missing, stop and tell the user exactly what to install — do not attempt
  to create a venv with an incompatible interpreter.

## Step 2 — Check for an existing `.venv`
- If a `.venv/` directory already exists, ask the user whether to reuse it,
  recreate it from scratch, or abort — never delete an existing venv
  without confirmation.

## Step 3 — Create the virtual environment
```bash
python3 -m venv .venv
```

## Step 4 — Give the correct activation command for the user's shell
- macOS / Linux (bash/zsh): `source .venv/bin/activate`
- Windows (PowerShell): `.venv\Scripts\Activate.ps1`
- Windows (cmd.exe): `.venv\Scripts\activate.bat`

Detect the OS from context if possible (e.g. file paths already seen in the
conversation); otherwise ask the user which shell they're using rather than
assuming.

## Step 5 — Install project dependencies
Inside the activated venv, run:
```bash
make install
```
This runs `pip install -e ".[dev]"` and `pre-commit install` as defined in
the `Makefile`. Do not reimplement these steps by hand — always go through
`make install` so behavior stays consistent with CI.

> If this step requires adding a *new* dependency that isn't already in
> `pyproject.toml` (e.g. the user asks to also install a library while
> setting up), that's a design change — follow `.clinerules/05-documentation.md`
> and log it in `CHANGELOG.md`, plus an ADR if the choice of library is
> non-trivial.

## Step 6 — Verify the environment works
```bash
make check
```
Run this and show the actual output. A working setup means lint, typecheck,
and the full test suite all pass. If anything fails here on a fresh
checkout (not because of a code bug, but an environment issue — e.g. a
missing system library), report the exact error and do not silently
continue as if setup succeeded.

## Reporting back
Confirm the Python version used, the activation command for the user's
shell, and the final `make check` result. Do not claim the environment is
ready without having actually run and observed Step 6 in this session.
