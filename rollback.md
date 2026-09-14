# Rollback Workflow

This workflow handles cancellation of recent changes and reversion to a stable state. Invoke as `/rollback` in Cline chat or when emergency revert needed. Follow steps **in order**:

## Step 0 — Verify rollback necessity

- Is this a genuine emergency (production broken, tests failing unexpectedly)?
- Or is this a code quality issue that should be fixed forward instead?
- Rollback is for emergencies; fixes are preferred for normal development.

## Step 1 — Identify rollback target

- **Last stable commit:** `git log --oneline -5` to find previous good state
- **Last tagged release:** `git tag -l` to find stable version (e.g., `v1.2.3`)
- **Previous deployment:** Find exact commit hash from deployment logs
- **Scope:** Is rollback code-only? Or does it include DB/infrastructure?

## Step 2 — Perform rollback (choose one method)

### Method A: Revert to previous commit (creates new commit)
```bash
git log --oneline -10  # Find commit to rollback to
git revert <commit-hash>  # Creates new commit that undoes changes
git push origin <branch-name>
```

**Advantage:** Preserves history. Creates audit trail.

**Disadvantage:** Slower (new commit added).

### Method B: Reset to tagged release (clean history)
```bash
git log --oneline -10  # Find tag
git checkout <tag-name>  # Detach at tag
git checkout -b rollback-<tag-name>  # Create branch from tag
git push origin rollback-<tag-name>
```

Then create PR to merge rollback branch to main for review.

**Advantage:** Clean, explicit revert point.

**Disadvantage:** Requires PR review cycle (slower).

### Method C: Hard reset (destructive — requires confirmation)
```bash
git reset --hard <commit-hash>  # WARNING: Loses commits after this point
git push origin <branch-name> --force-with-lease
```

**Use only if:** Commits after target are invalid/unwanted entirely.

**Risk:** Irreversible locally; `--force-with-lease` prevents accidental clobber of remote.

## Step 3 — Validate rollback

- Run `make test` to confirm codebase is stable post-rollback.
- Run `make lint` and `make typecheck` to confirm no style issues.
- If using database/integration rollback: verify snapshot/state is correct.
- If tests fail: do NOT push. Revert rollback and diagnose.

## Step 4 — Document rollback reason

Create issue in tracker: `"Post-Mortem: Rollback on [date]"`

Record:
- What went wrong (symptom, root cause if known)
- What tests were missing (why wasn't this caught?)
- Mitigation (new tests, new checks, process changes)
- When/if to re-attempt fix

Update `CHANGELOG.md` under `## [Unreleased] - Removed` or `Changed`:
```
- Removed: Feature X (rolled back 2026-09-14 due to critical bug in auth)
```

## Step 5 — Fix and re-deploy

- Fix root cause in **new feature branch** (do not push direct to main).
- Add regression test covering the original bug.
- Follow normal TDD workflow (`new-feature` or `fix-bug` workflow).
- Merge via PR for review.
- Tag release once tests pass.

## Anti-Patterns

- ❌ **Force push without review:** Loses history. Use `git revert` (audit trail) or PR-based rollback.
- ❌ **Rollback without investigation:** Fix forward instead. Document why revert was necessary.
- ❌ **No tagged releases:** Can't easily rollback to "stable". Tag every release.
- ❌ **Manual rollback without automation:** Error-prone. Script via GitHub Actions or similar if frequent.

## Automated Rollback (GitHub Actions)

If rollbacks are frequent, add this workflow to `.github/workflows/rollback.yml`:

```yaml
name: Emergency Rollback

on:
  workflow_dispatch:
    inputs:
      rollback_target:
        description: 'Commit hash or tag to rollback to'
        required: true
      reason:
        description: 'Reason for rollback'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Verify rollback target exists
        run: git rev-parse ${{ github.event.inputs.rollback_target }}

      - name: Create rollback commit
        run: |
          git config user.name "CI Rollback"
          git config user.email "ci@example.com"
          git revert --no-edit HEAD
          git push origin main

      - name: Log rollback event
        run: |
          echo "Rolled back to: ${{ github.event.inputs.rollback_target }}"
          echo "Reason: ${{ github.event.inputs.reason }}"
          echo "Time: $(date -u +'%Y-%m-%dT%H:%M:%SZ')"
```

**Trigger manually via GitHub UI:**
- Actions → Emergency Rollback → "Run workflow"
- Enter commit hash/tag and reason
- Confirm

## Reporting back

After rollback completes:
1. State which method was used (revert / reset / automated).
2. Show `make test` output confirming stability.
3. Link to post-mortem issue.
4. State next step (fix forward in new branch).
