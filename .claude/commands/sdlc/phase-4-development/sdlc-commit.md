# sdlc-commit — SDLC Phase 4: Development

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/plans/plan-{storyKey}.md` — implementation plan (summary of changes)
- `CLAUDE.md` (if exists) — project commit conventions

## Input
$ARGUMENTS — A Jira story key (e.g. `PROJ-010`)

## Task
Generate a conventional commit message linked to the Jira ticket, stage and commit all changes, and produce a PR description that traces back to the Jira story.

### Step 1: Gather Change Context
Run the following to understand what changed:
```bash
git status                   # See which files are staged/unstaged
git diff --stat HEAD         # Summary of file changes
git diff HEAD                # Full diff of all changes
git log --oneline -10        # Recent commit history (to match style)
```

Also read the plan file at `.claude/SDLCs/plans/plan-{storyKey}.md` for the story title and phase summaries.

### Step 2: Fetch Story Title from Jira
```
GET $JIRA_BASE_URL/rest/api/3/issue/{storyKey}
Authorization: Basic base64($JIRA_EMAIL:$JIRA_API_TOKEN)
```
Use the `summary` field as the commit description base.

### Step 3: Determine Commit Type
Select the appropriate conventional commit type based on the nature of the changes:
- `feat` — new feature or user-visible behavior (most stories)
- `fix` — bug fix
- `refactor` — code restructuring without behavior change
- `test` — adding or fixing tests only
- `docs` — documentation only
- `chore` — build/config/tooling changes
- `perf` — performance improvement

### Step 4: Compose the Commit Message
Follow conventional commits format with Jira ticket reference:

```
{type}({storyKey}): {concise description in imperative mood}

{Optional: 1–3 sentence body explaining WHY, not WHAT}

Refs: {storyKey}
{If breaking change:} BREAKING CHANGE: {description}
```

Rules:
- Subject line ≤ 72 characters
- Use imperative mood: "add", "implement", "fix" — not "added", "implementing"
- Body explains motivation, not the diff
- Always include `Refs: {storyKey}` footer

Example:
```
feat(PROJ-010): implement real-time document sync via WebSocket

Adds presence tracking and conflict resolution using OT (Operational
Transformation) to support concurrent edits without data loss.

Refs: PROJ-010
```

### Step 5: Stage and Commit
Stage all modified files relevant to the story:
```bash
git add {file1} {file2} ...  # Add specific files (not git add -A)
git commit -m "{commit message}"
```

If there are test files, stage them too. Do not stage unrelated files.

### Step 6: Generate PR Description
Write a pull request description using this template:

```markdown
## Summary
Implements [{storyKey}]({JIRA_BASE_URL}/browse/{storyKey}): {story title}

### Changes
{Bulleted list of key changes, one per implementation phase}

### Testing
- [ ] Unit tests added/updated (run: `bun test`)
- [ ] Integration tests pass (run: `bun test --integration`)
- [ ] Manual testing checklist:
  - [ ] {specific scenario from acceptance criteria 1}
  - [ ] {specific scenario from acceptance criteria 2}

### Acceptance Criteria
{Paste acceptance criteria from the Jira story}

### Screenshots / Recordings
{If UI changes: add screenshots here}

---
**Jira:** [{storyKey}]({JIRA_BASE_URL}/browse/{storyKey})
**Epic:** [{epicKey}]({JIRA_BASE_URL}/browse/{epicKey})
```

Print the PR description to the terminal so the user can copy it.

### Step 7: Jira Sync
Add a comment to the Jira story with the commit SHA and branch name:
```
POST $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Committed: {commit_sha_short} on branch {branch_name}. PR description generated." }
    ]}]
  }
}
```

## Output
1. **Git commit:** Staged and committed with conventional commit message
2. **PR description:** Printed to terminal, ready to paste into GitHub/Bitbucket/GitLab
3. **Jira action:** Commit SHA added as comment on the story
4. **Summary:** Commit hash, branch name, and list of files committed

## Validation
- [ ] Commit message follows `{type}({storyKey}): {description}` format
- [ ] Subject line ≤ 72 characters
- [ ] `Refs: {storyKey}` footer present
- [ ] All story-related files are committed (no relevant unstaged changes)
- [ ] PR description includes Jira link and acceptance criteria
- [ ] Comment added to Jira story

## Next Step
After completion, suggest: `/sdlc-test {storyKey}`
