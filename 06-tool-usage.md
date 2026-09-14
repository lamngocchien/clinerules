# Tool Usage & Avoiding Edit Loops

## Never retry a failed edit identically
If a file edit (diff/patch-based `old_str` match, or any similar
find-and-replace edit) fails, do NOT resend the exact same call. Instead:

1. Re-read the file's current full content first — never assume it still
   matches what you last saw, especially if you or another step already
   edited it earlier in this task.
2. Diagnose why the match failed before retrying: whitespace/indentation
   mismatch, the target text appearing more than once, or the surrounding
   code having changed.
3. Retry with a corrected match based on what you just read — widen the
   matched context if it wasn't unique, or fix whitespace if that was the
   issue.

## Cap retries — escalate instead of looping
- Allow at most **2 attempts** at fixing the same failed edit. If it still
  fails on the second attempt, stop and explain to the user exactly what's
  blocking it (e.g. "the text I'm trying to match appears twice in the
  file" or "the file's indentation uses tabs, not spaces, which is throwing
  off my match") — do not keep retrying with no new information.
- Never issue more than 2 consecutive identical tool calls of any kind. If
  a second identical call would be made, stop and ask the user for
  guidance instead.

## Prefer full-file rewrite when uncertain
- For files under ~150 lines with simple structure (single function/class), if a targeted patch has failed once, prefer rewriting the entire file with the corrected content over attempting another partial diff — it removes the ambiguity that caused the failure.
- For larger or complex files (multiple classes, nested logic), narrow the edit to a smaller, more specific, and more clearly unique chunk of surrounding text rather than retrying the same large match.
- Always re-read the file before deciding whether to retry or rewrite — do not guess based on memory of earlier state.

## After any edit, verify before moving on
- After an edit succeeds, briefly re-read the changed section to confirm it
  matches intent before running tests — do not chain multiple edits to the
  same file without checking each one landed correctly.

## Commit message format (conventional commits)
- Format: `<type>(<scope>): <subject>` with optional body and footer.
- **Type:** `feat` (new feature), `fix` (bug fix), `refactor` (code restructure without behavior change), `test` (test-only changes), `docs` (documentation only), `chore` (tooling, deps, config).
- **Scope** (optional): affected module name (e.g., `auth`, `user-service`). Omit if change affects entire project.
- **Subject:** lowercase, imperative mood ("add feature" not "added feature"), no period, under 50 characters.
- **Body** (optional): detailed explanation of why the change was made, not what changed. Wrap at 72 characters.
- **Example:** `feat(auth): add JWT token refresh endpoint` or `fix: resolve race condition in cache invalidation`

## Pre-commit hooks

- Project includes pre-commit hooks (configured in `.pre-commit-config.yaml` if present). Hooks run `ruff`, `black`, and `mypy --strict` before each commit.
- Install hooks: `pre-commit install` (one-time setup).
- Run manually: `pre-commit run --all-files` to validate all files without committing.
- Hooks auto-fix formatting issues; if a hook fails, review the changes, stage, and commit again.
