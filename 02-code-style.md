# Code Style Standards

- Strictly follow PEP 8, enforced via `ruff` + `black` (line-length 100,
  already configured in `pyproject.toml`).
- **Naming conventions:**
  - Public functions/methods: `snake_case` (e.g., `calculate_total()`).
  - Private functions/methods: prefix with single underscore (e.g., `_helper_function()`). Not enforced by Python but signals "internal, don't rely on this."
  - Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES = 3`). Use only for immutable module-level values.
  - Classes: `PascalCase` (e.g., `UserRepository`).
- Every public function/method/class must have **full type hints**
  (parameters + return value). The project runs `mypy --strict`; do not use
  `Any` to dodge type-checking unless truly necessary, with a comment
  explaining why.
- Google-style docstrings for every module, class, and public function:

  ```python
  def divide(a: float, b: float) -> float:
      """Return a divided by b.

      Args:
          a: Numerator.
          b: Denominator.

      Returns:
          The result of a / b.

      Raises:
          DivisionByZeroError: If b is zero.
      """
  ```

- Always start new modules with `from __future__ import annotations`. This enables postponed evaluation of type hints (PEP 563), allowing forward references and reducing import cycles. Place it as the first import, before any other statements or docstrings.
- Use clear, descriptive names; prefer a longer name that expresses intent
  over a short, cryptic abbreviation.
- Avoid mutable global state; prefer dependency injection through function
  or constructor parameters to keep code testable.
- Custom exceptions must inherit from an appropriate Python stdlib exception
  (e.g. `ValueError`, `RuntimeError`), be named ending in `Error`, and have a
  docstring describing when they are raised.
- **Exception handling:** Never use bare `except:` or catch `Exception` broadly without logging/re-raising. Always catch specific exception types (e.g., `except ValueError:`, `except (IOError, OSError):`). If you must catch broad exceptions, document why and log before re-raising.
- **Circular imports:** Use `TYPE_CHECKING` guard at module top for type-only imports that would otherwise cause circular dependencies:
  ```python
  from __future__ import annotations
  from typing import TYPE_CHECKING
  if TYPE_CHECKING:
      from app.other_module import SomeClass  # Import only during type-checking
  ```
- **Method naming (private/protected/dunder):** Private methods (`_method`) signal "internal, don't call directly" but are not strictly enforced; protected methods (rare in Python) have no special prefix. Dunder methods (`__method__`) are reserved for Python magic (e.g., `__init__`, `__str__`). Do not use dunder for custom "private" methods — use single underscore instead.
- Import order: stdlib → third-party → first-party (`app.*`), with a blank
  line between groups — `ruff` (isort) enforces this automatically.
