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

---

## Project Documentation Standards

This section defines when and how to update `README.md` and `requirements.txt` in the `src/` directory. These files are project-facing documentation and must stay synchronized with code changes.

### README.md (`src/README.md`)

**Purpose:** User-facing guide covering features, installation, usage examples, project structure, and development workflow.

#### Update triggers (add entry or modify section)

| Situation | Action |
|-----------|--------|
| New user-visible feature added (GUI button, calculation method, etc.) | Add to "Features" section with brief description |
| Feature behavior changes (e.g., output format, error handling) | Update relevant feature description in-place |
| New module added to `src/app/` | Add to "Project Structure" diagram |
| Installation instructions change | Update "Installation" and "Setup" sections |
| New keyboard shortcut or UI interaction | Add to "GUI" feature bullet |
| Test command or development workflow changes | Update "Testing" and "Development" sections |

#### Style

- Keep descriptions concise (1–2 lines per feature).
- Include code examples for module usage (Python REPL style).
- Link to `.clinerules/` for governance details, not inline explanations.
- Use markdown with clear hierarchy (H2 for sections, H3 for subsections).
- Never remove past features or sections — if deprecated, add deprecation note.

#### When to commit

- Changes to README.md that accompany code changes use the code-related commit prefix (e.g., `feat:` if a feature was added).
- If only README.md was updated (no code), use `docs: update README.md for <reason>`.

---

### requirements.txt (`src/requirements.txt`)

**Purpose:** Pin exact runtime dependencies and their versions. Separate from `pyproject.toml` dev dependencies for clarity.

#### Format

```
# Comments explain each dependency

package-name>=1.2.3,<2.0  # Reason or usage
another-package==1.5.0     # Pinned for compatibility
```

- One package per line.
- Use `==` for pinned versions (immutable, reproducible).
- Use `>=X.Y,<Z` for ranges (only if version flexibility is intentional).
- Add inline comments explaining why each dependency is needed.
- Keep file short — add only runtime production dependencies, not dev-only tools.

#### Update triggers

| Situation | Action |
|-----------|--------|
| New runtime package imported in `src/app/` code | Add to requirements.txt with pinned version |
| Package version bump for security/bug fix | Update version constraint in-place |
| Package no longer used | Remove line (with brief comment in git log) |
| Conditional dependency (e.g., "GUI optional") | Add comment explaining condition; comment-out if not default |

#### Validation

- After editing requirements.txt, verify it installs cleanly:
  ```bash
  pip install -r src/requirements.txt
  ```
- Run `make test` to confirm all imports still resolve.

#### When to commit

- Changes to requirements.txt with new feature: include in same commit as code (e.g., `feat(gui): add flet dependency`).
- Dependency version bump for security: `chore(deps): bump package-name to 1.2.3`.

---

### Synchronization Rules

1. **Every feature that adds an import** → update both files:
   - README.md: add feature description
   - requirements.txt: add package + version

2. **Every breaking change to existing feature** → update README.md immediately (examples, behavior description).

3. **100% coverage check:** Code changes → update docs → verify tests still pass → commit as atomic unit.

4. **No docs debt:** If a README section is wrong or stale after a code change, fix it in the same task—do not defer to "later."

5. **Preserve history:** Never delete past sections from README.md; use deprecation notes instead. Keep requirements.txt clean (remove unused packages).

---

### Anti-Patterns for Project Documentation

- ❌ Add feature to code, skip README → confuses users, creates debt.
- ❌ Pin version to `*` or leave blank → loses reproducibility.
- ❌ Import new package without adding to requirements.txt → breaks fresh installs.
- ❌ Update README.md with speculative future features → document only what's implemented.
- ❌ Commit code without updating requirements.txt, then fix in next commit → creates breakage window.

---

### Review Checklist

Before marking a feature task complete:

- [ ] Code change passes all tests (100% coverage).
- [ ] README.md updated with new feature description (if user-facing) or usage example (if module).
- [ ] requirements.txt updated if new packages were imported.
- [ ] Lint (`make lint`) and type check (`make typecheck`) pass.
- [ ] Commit message uses conventional format (`feat:`, `fix:`, `docs:`, etc.).
- [ ] Git log is clean — no intermediate "fix README" or "add dep" commits that should be squashed.
