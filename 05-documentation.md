# Documentation & Design Record Keeping

Every change that adds, modifies, or removes a feature must update the
project's living documentation as part of the same task — not as an
afterthought, and not only when explicitly asked. Treat this as part of
"done," the same way passing tests are part of "done."

## What to update, and when

| Situation | File to update | How |
|---|---|---|
| Any user-visible change (feature, fix, behavior change) | `CHANGELOG.md` | Add a bullet under `## [Unreleased]`, in the correct section (`Added` / `Changed` / `Fixed`). Never edit or remove a past entry. |
| A change alters module responsibilities, adds/removes a module, or changes how data flows through the system | `docs/architecture.md` | Edit the relevant section **in place** so it always reflects current reality — this file has no history, it only ever describes "now." |
| A non-obvious technical decision was made (library choice, trade-off, constraint imposed by the user) | New file in `docs/decisions/000N-*.md` | Copy `docs/decisions/0000-template.md`, fill it in, use the next sequential number. Never edit a past ADR's Decision/Context — if a decision is reversed, write a new ADR and mark the old one `Status: Superseded by ADR-000M`. |

## Rules

- These updates are part of the REFACTOR step (or immediately after GREEN,
  before reporting the task complete) — never skip them because the tests
  are already green.
- **Commit message for documentation updates:** If a task includes only documentation changes (no code changes), use commit prefix `docs:` (e.g., `docs: update architecture.md for new caching layer`). If code and documentation both change, use the code-related prefix (e.g., `feat: add user service`, which implicitly includes updated CHANGELOG and architecture docs).
- `docs/architecture.md` must never contain information that is no longer
  true. If unsure whether a section is still accurate after a change,
  re-read the current code before editing it, don't guess.
- When a requirement comes from the user in chat (not from an existing doc),
  and it materially shapes the design, capture the *reason* in an ADR — the
  chat log is not durable project memory, the docs are.
  - **Material decision criteria:** Write an ADR if the decision affects 2+ modules, imposes a long-term constraint (e.g., "we must use async/await everywhere"), reverses a past decision, involves a significant library choice, or defines non-obvious architecture (e.g., how caching layers interact). Do NOT write an ADR for local implementation details ("use dict comprehension here") or style choices ("name this variable `x_val`").
- Before implementing a new feature, check `docs/architecture.md` and any
  relevant ADRs first, so the new work is consistent with existing
  decisions instead of contradicting them silently. If the new request
  conflicts with a past ADR, say so explicitly and ask whether to supersede
  it — do not silently override a documented decision.
- Do not create an ADR for every small change — see the criteria in
  `docs/decisions/0000-template.md`. Over-documenting trivial choices makes
  the real decisions harder to find.

## Deprecation pattern

When removing or replacing a function, class, or module:
1. Mark with `@deprecated` decorator or docstring note: `.. deprecated:: <version> Use <replacement> instead.`
2. Add entry to `CHANGELOG.md` under `## [Unreleased] - Deprecated` with migration path.
3. Keep deprecated code working for at least one major version; log a warning when called (e.g., `warnings.warn("function_name is deprecated, use new_function instead", DeprecationWarning)`).
4. Write a test that the deprecation warning is raised (use `pytest.warns(DeprecationWarning)`).
5. When removing in a future release, update CHANGELOG under `## [X.Y.Z] - Removed` and reference the deprecation version.
