# Rules Audit Workflow

This workflow is invoked as `/audit-rules`. Use it right after switching the
underlying LLM model, or periodically, to verify the rules are actually
being loaded and followed — not just present on disk.

## Step 1 — Prove the files were read
List every file under `.clinerules/` (including `.clinerules/workflows/`)
that is currently loaded in context. For each one, give a ONE-sentence
summary of its core rule, in your own words — not copy-pasted from the
file. If a file is missing from this list, say so explicitly; do not
silently omit it.

## Step 2 — Self-check against the last completed task
Look at the most recent code change made in this conversation (or ask the
user to point to one). Check it against each rule file and report,
file-by-file:
- Was this rule followed? (yes / no / not applicable to that change)
- If "no," say so plainly — do not rationalize a violation as compliant.

## Step 3 — Report context health
State whether anything suggests truncation or partial loading — e.g. if
asked to quote a specific line from a `.clinerules` file and it cannot be
produced accurately, or if a file's content seems inconsistent with what is
in the repository. If context feels degraded, tell the user to try:
- Starting a fresh Cline task/session (resets context).
- Checking Cline's settings for the currently selected model and confirming
  it has enough context window for the project's `.clinerules` total size.
