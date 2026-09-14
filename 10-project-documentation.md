# Project Documentation Standards

This document defines when and how to update `README.md` and `requirements.txt` in the `src/` directory. These files are project-facing documentation and must stay synchronized with code changes.

## README.md (`src/README.md`)

**Purpose:** User-facing guide covering features, installation, usage examples, project structure, and development workflow.

### Update triggers (add entry or modify section)

| Situation | Action |
|-----------|--------|
| New user-visible feature added (GUI button, calculation method, etc.) | Add to "Features" section with brief description |
| Feature behavior changes (e.g., output format, error handling) | Update relevant feature description in-place |
| New module added to `src/app/` | Add to "Project Structure" diagram |
| Installation instructions change | Update "Installation" and "Setup" sections |
| New keyboard shortcut or UI interaction | Add to "GUI" feature bullet |
| Test command or development workflow changes | Update "Testing" and "Development" sections |

### Style

- Keep descriptions concise (1–2 lines per feature).
- Include code examples for module usage (Python REPL style).
- Link to `.clinerules/` for governance details, not inline explanations.
- Use markdown with clear hierarchy (H2 for sections, H3 for subsections).
- Never remove past features or sections — if deprecated, add deprecation note.

### When to commit

- Changes to README.md that accompany code changes use the code-related commit prefix (e.g., `feat:` if a feature was added).
- If only README.md was updated (no code), use `docs: update README.md for <reason>`.

---

## requirements.txt (`src/requirements.txt`)

**Purpose:** Pin exact runtime dependencies and their versions. Separate from `pyproject.toml` dev dependencies for clarity.

### Format

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

### Update triggers

| Situation | Action |
|-----------|--------|
| New runtime package imported in `src/app/` code | Add to requirements.txt with pinned version |
| Package version bump for security/bug fix | Update version constraint in-place |
| Package no longer used | Remove line (with brief comment in git log) |
| Conditional dependency (e.g., "GUI optional") | Add comment explaining condition; comment-out if not default |

### Validation

- After editing requirements.txt, verify it installs cleanly:
  ```bash
  pip install -r src/requirements.txt
  ```
- Run `make test` to confirm all imports still resolve.

### When to commit

- Changes to requirements.txt with new feature: include in same commit as code (e.g., `feat(gui): add flet dependency`).
- Dependency version bump for security: `chore(deps): bump package-name to 1.2.3`.

---

## Synchronization Rules

1. **Every feature that adds an import** → update both files:
   - README.md: add feature description
   - requirements.txt: add package + version

2. **Every breaking change to existing feature** → update README.md immediately (examples, behavior description).

3. **100% coverage check:** Code changes → update docs → verify tests still pass → commit as atomic unit.

4. **No docs debt:** If a README section is wrong or stale after a code change, fix it in the same task—do not defer to "later."

5. **Preserve history:** Never delete past sections from README.md; use deprecation notes instead. Keep requirements.txt clean (remove unused packages).

---

## Anti-Patterns

- ❌ Add feature to code, skip README → confuses users, creates debt.
- ❌ Pin version to `*` or leave blank → loses reproducibility.
- ❌ Import new package without adding to requirements.txt → breaks fresh installs.
- ❌ Update README.md with speculative future features → document only what's implemented.
- ❌ Commit code without updating requirements.txt, then fix in next commit → creates breakage window.

---

## Review Checklist

Before marking a feature task complete:

- [ ] Code change passes all tests (100% coverage).
- [ ] README.md updated with new feature description (if user-facing) or usage example (if module).
- [ ] requirements.txt updated if new packages were imported.
- [ ] Lint (`make lint`) and type check (`make typecheck`) pass.
- [ ] Commit message uses conventional format (`feat:`, `fix:`, `docs:`, etc.).
- [ ] Git log is clean — no intermediate "fix README" or "add dep" commits that should be squashed.
