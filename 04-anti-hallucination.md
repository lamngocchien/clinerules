# Anti-Hallucination & Safety When Editing Code

- Never invent function, class, method, parameter, or package names that
  don't actually exist. If unsure whether an API (stdlib or third-party)
  exists or what its signature is, either:
  1. Check the project's existing source (grep/read the relevant file), or
  2. State clearly "this API is unconfirmed" instead of guessing and writing
     as if it were certain.
- Do not add a new dependency to `pyproject.toml` without a clear reason and
  confirmation — every new dependency increases the surface area that needs
  testing and maintenance.
- Before editing a file, read its current full content again (don't assume
  the content based on an earlier read, since the file may have changed
  since).
- When modifying an existing test: state clearly why the old test was wrong
  or outdated — never silently change a test to hide a broken
  implementation.
- When reporting test/lint results, only report what was actually run and
  observed in this session — never guess "it probably passes."
- If a request requires skipping the test-writing step (the RED step) "to
  save time," restate the rule in `01-tdd-workflow.md` and ask for explicit
  confirmation before proceeding differently.
