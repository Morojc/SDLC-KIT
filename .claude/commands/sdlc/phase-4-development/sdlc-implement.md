# sdlc-implement — SDLC Phase 4: Development

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/plans/` — implementation plans
- `.claude/SDLCs/designs/` — design documents for architectural context
- `CLAUDE.md` (if exists) — project conventions, coding standards, patterns

## Input
$ARGUMENTS — A plan file path (e.g. `plan-PROJ-010.md`) or a Jira story key (e.g. `PROJ-010`)

## Task
Execute the implementation plan phase by phase, writing code to the specified file targets, validating after each phase, and logging progress on the Jira sub-tasks.

### Step 1: Load the Plan
If the argument is a file path, read `.claude/SDLCs/plans/{argument}`.
If the argument is a Jira key, read `.claude/SDLCs/plans/plan-{storyKey}.md`.

Parse out:
- All implementation phases (title, files, dependencies, parallel flag, estimates)
- Validation commands per phase
- Jira story key and sub-task keys

### Step 2: Pre-flight Checks
Before writing any code:
1. Read every file listed in **File Targets** that already exists (to understand existing patterns)
2. Read relevant test files to understand the testing framework and conventions
3. Confirm the validation commands work on the current codebase (run them once as a baseline)
4. Note the current git status (`git status`) — implementation should only touch plan-scoped files

### Step 3: Execute Phases Sequentially
For each implementation phase (respecting the dependency order in the plan):

**3a. Implement the phase:**
- Write or modify the files listed for this phase
- Follow existing code conventions (indentation, import order, naming, error handling)
- Write clean, minimal code — no gold-plating, no premature abstraction
- Add tests alongside implementation code

**3b. Validate the phase:**
Run the validation command(s) specified for this phase. If they fail:
- Diagnose the failure
- Fix the implementation
- Re-run validation until it passes
Do not proceed to the next phase until the current phase passes its validation gate.

**3c. Log time on the Jira sub-task:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{subTaskKey}/worklog
Authorization: Basic base64($JIRA_EMAIL:$JIRA_API_TOKEN)
{
  "timeSpentSeconds": {estimated_seconds},
  "comment": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Phase complete: {phase title}. Validation passed." }
    ]}]
  }
}
```

**3d. Transition the sub-task to Done:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{subTaskKey}/transitions
{ "transition": { "id": "{done_transition_id}" } }
```

### Step 4: Run Full Validation Suite
After all phases are complete, run the complete validation suite:
```bash
bun test           # All tests must pass
bun run lint       # Zero lint errors
bun run typecheck  # No TypeScript errors
bun run build      # Build succeeds (if applicable)
```

If any check fails, fix the issue before proceeding.

### Step 5: Update the Plan File
Update `.claude/SDLCs/plans/plan-{storyKey}.md` — change each phase's `Status` column from `pending` to `done`.

### Step 6: Jira Sync
**6a. Add an implementation summary comment to the story:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Implementation complete. {N} phases executed, all tests passing. Ready for commit and code review." }
    ]}]
  }
}
```

**6b. (Optional) Attach a brief summary of changed files as a comment:**
List files created/modified and the tests that cover them.

## Output
1. **Code changes:** All files listed in the plan's File Targets, created/modified
2. **Updated plan:** `.claude/SDLCs/plans/plan-{storyKey}.md` with all phases marked done
3. **Jira actions:**
   - Time logged on each sub-task
   - All sub-tasks transitioned to Done
   - Implementation summary comment on the story
4. **Summary:** Print full validation results (test count, coverage, lint status)

## Validation
- [ ] All implementation phases complete (no `pending` phases remain in plan file)
- [ ] `bun test` passes (or equivalent)
- [ ] `bun run lint` passes with zero errors
- [ ] `bun run typecheck` passes with no errors
- [ ] All Jira sub-tasks are Done
- [ ] Implementation comment added to story

## Next Step
After completion, suggest: `/sdlc-commit {storyKey}`
