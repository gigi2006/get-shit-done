<purpose>
Cleanly abort a phase that's in a bad state (failed executor, broken plans, stuck execution) and reset to a clean state. Provides three recovery strategies: rollback commits, keep commits but reset plans, or full directory reset. Updates STATE.md and offers clear next steps.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

<step name="init_context">
Load current project state:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init)
```

Extract: `project_root`, `current_phase`, `current_plan`, `total_plans`.

Also read STATE.md and the current phase directory contents to understand what exists.

If no active phase is detected:

```
ERROR: No active phase found in STATE.md

Nothing to abort. Use /gsd:progress to check project status.
```

Exit.
</step>

<step name="gather_phase_status">
**Collect what happened in this phase so far:**

1. **Completed plans**: List all SUMMARY.md files in the phase directory (each = one completed plan)
2. **Current plan**: Read PLAN.md if it exists — show task status
3. **Commits made**: Find all commits associated with this phase since it started

```bash
# List completed plan summaries
ls .planning/phases/${PHASE_DIR}/SUMMARY*.md 2>/dev/null

# Find commits made during this phase (look for phase name in commit messages)
git log --oneline --since="$(git log --format='%aI' --diff-filter=A -- .planning/phases/${PHASE_DIR}/ | tail -1)" -- . 2>/dev/null
```

4. **Uncommitted changes**: Check `git status` for dirty working tree
5. **Files modified**: List files changed during this phase

Present a summary:

```
Phase {N}: {Name} — Abort Assessment

Status:
- Plans completed: {X} of {Y}
- Current plan: {Z} (tasks {done}/{total})
- Commits in this phase: {count}
- Uncommitted changes: {yes/no}

Commits made during this phase:
  {hash1} {message1}
  {hash2} {message2}
  ...
```
</step>

<step name="choose_strategy">
**Ask the user which abort strategy to use:**

```
How do you want to abort Phase {N}?

1. ROLLBACK COMMITS
   Revert all {count} commits made during this phase.
   Code returns to pre-phase state. Plans are removed.
   Use when: The entire phase approach was wrong.

2. KEEP COMMITS, RESET PLANS
   Keep all code changes but mark plans as incomplete.
   Phase can be re-planned from scratch with code intact.
   Use when: Code is fine but planning/execution went off track.

3. FULL RESET
   Remove all phase directory contents and start fresh.
   Commits stay in history but phase planning is wiped clean.
   Use when: Plans are broken/corrupt and need a clean slate.

Choose (1/2/3):
```

Wait for user response.
</step>

<step name="handle_uncommitted">
**Before any strategy, handle uncommitted changes:**

If there are uncommitted changes in the working tree:

```
WARNING: You have uncommitted changes.

Modified files:
  {file1}
  {file2}

These must be handled before aborting:
  a) Stash them (can be restored later)
  b) Discard them (permanent)
  c) Cancel abort

Choose (a/b/c):
```

- **Stash**: `git stash push -m "abort-phase-{N}: uncommitted work"`
- **Discard**: `git checkout -- .` (confirm twice — destructive)
- **Cancel**: Exit workflow

Wait for confirmation before proceeding.
</step>

<step name="execute_rollback" condition="strategy=1">
**Strategy 1: Rollback Commits**

List each commit to be reverted and confirm:

```
The following commits will be reverted (newest first):

  {hashN} {messageN}
  ...
  {hash1} {message1}

This will create {count} revert commits. Proceed? (y/n)
```

Wait for confirmation, then revert in reverse chronological order:

```bash
# Revert each commit (newest first, no auto-edit)
git revert --no-edit {hashN}
git revert --no-edit {hashN-1}
# ... etc
```

If any revert has conflicts:

```
CONFLICT during revert of {hash}: {message}

Options:
  a) Resolve manually (abort workflow pauses here)
  b) Abort the entire rollback (restore to pre-rollback state)

Choose (a/b):
```

After successful reverts, clean up phase directory:

```bash
# Remove phase plan files (keep directory for STATE.md reference)
rm -f .planning/phases/${PHASE_DIR}/PLAN.md
rm -f .planning/phases/${PHASE_DIR}/SUMMARY*.md
rm -f .planning/phases/${PHASE_DIR}/.continue-here.md
```
</step>

<step name="execute_keep_commits" condition="strategy=2">
**Strategy 2: Keep Commits, Reset Plans**

```bash
# Remove plan artifacts but keep the phase directory
rm -f .planning/phases/${PHASE_DIR}/PLAN.md
rm -f .planning/phases/${PHASE_DIR}/SUMMARY*.md
rm -f .planning/phases/${PHASE_DIR}/.continue-here.md
```

The code changes from commits remain in place. The phase is ready to be re-planned.
</step>

<step name="execute_full_reset" condition="strategy=3">
**Strategy 3: Full Reset**

```
WARNING: This will delete ALL contents of:
  .planning/phases/${PHASE_DIR}/

Commits remain in git history but all planning files are gone.
Proceed? (y/n)
```

Wait for confirmation:

```bash
# Wipe phase directory contents
rm -rf .planning/phases/${PHASE_DIR}/*
rm -rf .planning/phases/${PHASE_DIR}/.*  2>/dev/null
```

Re-create an empty phase directory so STATE.md references remain valid:

```bash
mkdir -p .planning/phases/${PHASE_DIR}
```
</step>

<step name="update_state">
**Update STATE.md to reflect the abort:**

Modify STATE.md:
- Reset `current_plan` to 0
- Add a blocker note documenting the abort

The blocker note format:

```markdown
### Blockers
- Phase {N} aborted ({timestamp}) — Strategy: {rollback|keep-commits|full-reset}. Reason: [ask user for brief reason or use "phase in bad state"]
```

```bash
timestamp=$(node ~/.claude/get-shit-done/bin/gsd-tools.js current-timestamp full --raw)
```

If STATE.md already has a Blockers section, append to it. If not, add one.
</step>

<step name="commit_abort">
Stage and commit the abort changes:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "chore: abort phase {N} ({phase-name}) — {strategy}" --files .planning/
```
</step>

<step name="completion">
Present completion summary:

```
Phase {N} ({Name}) aborted.

Strategy: {description}
Changes:
- {what was done — reverts, file deletions, etc.}
- STATE.md updated (plan counter reset, blocker added)
- Committed: chore: abort phase {N} ({name}) — {strategy}

---

## What's Next

Would you like to:
- `/gsd:plan` — re-plan this phase from scratch
- `/gsd:skip-phase` — skip to the next phase
- `/gsd:progress` — review overall roadmap status
- `/gsd:pause-work` — stop working for now

---
```
</step>

</process>

<anti_patterns>

- Don't rollback commits without listing them and getting explicit confirmation
- Don't discard uncommitted changes without double confirmation (destructive)
- Don't delete the phase directory itself — only its contents (STATE.md references the directory)
- Don't modify ROADMAP.md — the phase still exists in the roadmap, it's just being reset
- Don't use `git reset --hard` — always use `git revert` to preserve history
- Don't abort a phase that has no work done (nothing to abort — inform the user)
- Don't silently skip conflict resolution during reverts

</anti_patterns>

<success_criteria>
Phase abort is complete when:

- [ ] Phase status fully assessed (plans, commits, uncommitted changes)
- [ ] User chose a strategy and confirmed destructive actions
- [ ] Uncommitted changes handled (stashed or discarded)
- [ ] Chosen strategy executed without errors
- [ ] STATE.md updated with reset plan counter and blocker note
- [ ] Changes committed with descriptive message
- [ ] User informed of result and next steps
</success_criteria>
