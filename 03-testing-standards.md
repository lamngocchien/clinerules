# Testing Standards

## Test structure
- One test file per module: `src/app/foo.py` → `tests/unit/test_foo.py`.
- Group tests under a `TestSubject` class when a function/class has multiple
  scenarios, or keep it flat for a handful of simple tests.
- Test names must describe the behavior, in the form
  `test_<subject>_<expected_behavior>`, e.g.
  `test_divide_by_zero_raises_domain_error`. Do not use generic names like
  `test_1`, `test_case_2`.
- Every test follows the **Arrange–Act–Assert** structure (blank lines may
  separate the three parts; comments are not required when the code is
  already clear).

## Categorizing tests with markers
- `@pytest.mark.unit` — **mandatory for all tests in `tests/unit/`**. No real I/O, fast (<100ms), every external dependency may be mocked. Tests in `tests/unit/` must explicitly carry this marker.
- `@pytest.mark.integration` — touches the real filesystem, network, DB, or subprocess. Lives in `tests/integration/`. **Boundary cases:** In-memory SQLite (`:memory:`) counts as unit (no real DB); mocked HTTP stubs count as unit (no real network); subprocess mocks count as unit.
- `@pytest.mark.slow` — a test known to be slow; excluded from the fast TDD loop via `make test-fast`.
- **Performance budgets:** Unit tests must complete in <100ms each (aggregate suite <30s). Integration tests <5s each (aggregate <60s). If a test exceeds budget, optimize or mark `@pytest.mark.slow` to exclude from `make test-fast`.
- **Parametrized tests:** Use `@pytest.mark.parametrize` to test multiple input/output pairs without code duplication. Each parameter combination counts as a separate test in coverage calculation. Example:
  ```python
  @pytest.mark.parametrize("input_val,expected", [(0, "zero"), (1, "one"), (-1, "negative")])
  def test_describe_number(input_val: int, expected: str) -> None:
      assert describe_number(input_val) == expected
  ```

## Mocking and fixtures
- Use `pytest-mock` (the `mocker` fixture) instead of `unittest.mock`
  directly, to keep mocking syntax consistent across the project.
- Only mock at system boundaries (network calls, filesystem, current time,
  subprocess) — never mock the internal logic of the module under test.
- Fixtures shared by 2+ modules go in `tests/conftest.py`; fixtures specific
  to a single module stay in that module's test file.
- **Fixture naming convention:** Name fixtures after the object they create (e.g., `user` for a User instance, `mock_api` for a mocked API client, `temp_file` for a temporary file). Prefix `mock_` or `mock` for mocked objects. Prefix `sample_` or `test_` only if multiple variations exist (e.g., `sample_user_admin`, `test_user_inactive`).
- **Fixture scope:** Use `scope="function"` (default) for most fixtures to ensure test isolation. Use `scope="module"` only for expensive, read-only resources (e.g., a fixture that builds a large in-memory data structure). Always verify that module-scoped fixtures do not carry state across tests.

## A test suite is not "done" until it's green
- Never report a task as complete unless `make test` passes 90%+ coverage and
  `make lint` / `make typecheck` are both clean.
- Flaky tests (occasionally failing without a code change) must be fixed
  immediately, not `@pytest.mark.skip`-ed to avoid dealing with them, unless
  there is a clearly documented reason and a follow-up plan in a comment.

## Coverage reporting & branch gaps
- Run `make test-cov` to generate a detailed coverage report and identify untested lines/branches.
- Report shows: line coverage (did every line execute?), branch coverage (did every if/else path execute?), and missing lines (which ones weren't hit).
- Use `--cov-report=term-missing` or open `htmlcov/index.html` after running tests to find gaps visually.
- Build fails if coverage drops below 90% (configured in `pyproject.toml`). Aim for 90%+ on all modules.
- Acceptable gaps (under 90% threshold): boilerplate, framework setup, platform-specific code. Mark with `# pragma: no cover` + explanation.

## Edge cases to consider for every new function
- Empty / None / boundary input (0, negative numbers, empty string, empty
  list).
- Invalid input → must raise the correct exception type, with an assertion
  on the message.
- For numeric functions: overflow, division by zero, negative numbers if
  the domain allows them.
- For string/collection functions: unicode, extra whitespace, duplicate
  elements.
