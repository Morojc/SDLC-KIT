---
name: sdlc-commit
description: "SDLC Phase 4: Create a conventional commit with Jira sync"
---

# sdlc-commit — SDLC Phase 4: Development

## MCP Tools
Prefer MCP tools over raw curl for all external operations:
- **Jira** (`jira` MCP): use `jira_create_issue`, `jira_update_issue`, `jira_add_comment`, `jira_transition_issue`, `jira_get_issue`, `jira_search_issues` — no curl to Jira API
- **GitHub** (`github` MCP): use `create_pull_request`, `create_branch`, `push_files`, `get_file_contents`, `search_code` — no curl to GitHub API


## Mulch & Seeds

### Session Start
```bash
mulch prime --context   # load project-specific expertise relevant to changed files
sd prime                # load issue tracking context and rules
```

### During Work
```bash
mulch search "<topic>"  # check prior decisions before implementing
sd update <id> --status in_progress  # claim your Seeds issue before writing code
```

### Session Close (run before saying "done")
```bash
mulch learn                          # review changed files for recordable insights
mulch record <domain> --type <type> --description "..."  # record conventions/decisions
mulch sync                           # validate and commit .mulch/ changes
sd close <id>                        # close completed Seeds issues
sd sync                              # sync Seeds with git
```


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

### Step 7: Open PR — Story Branch → Sprint Branch

Resolve branches:
- **Head (source):** `story/{storyKey}-{title-slug}` — the story's working branch
- **Base (target):** `sprint/{N}-{goal-slug}` — read from `.claude/SDLCs/stories/story-{storyKey}.md` `sprintBranch` field

```
# MCP: github — create_pull_request
# title: "{type}({storyKey}): {story title}"
# head:  "story/{storyKey}-{slug}"
# base:  "sprint/{N}-{goal-slug}"     ← sprint branch, NOT main
# body:  {PR description generated above}
```

**Important:** The PR targets the **sprint branch**, never `main` directly.
The sprint → main merge happens separately via `/sdlc-merge sprint/{N}` once all stories are done.

### Step 8: Jira Sync

```
# MCP: jira_add_comment
# Issue: {storyKey}
# Body: "Committed: {sha_short} on story/{storyKey}-{slug}
#        PR: {pr_url} → sprint/{N}-{goal-slug}
#        Status: In Review"
```

```
# MCP: jira_transition_issue
# Transition {storyKey} → "In Review"
```

## Output
1. **Git commit:** Staged and committed with conventional commit message
2. **PR opened:** `story/{storyKey}-{slug}` → `sprint/{N}-{slug}` (never directly to `main`)
3. **Jira:** Commit SHA + PR URL added as comment; story transitioned to In Review
4. **Summary:** Commit hash, story branch, sprint branch target, PR URL

## Validation
- [ ] Commit message follows `{type}({storyKey}): {description}` format
- [ ] Subject line ≤ 72 characters
- [ ] `Refs: {storyKey}` footer present
- [ ] All story-related files committed (no relevant unstaged changes)
- [ ] PR base is sprint branch — not `main`
- [ ] Jira story transitioned to In Review
- [ ] PR URL printed for user

## Next Step
When PR is reviewed and ready to merge:
`/sdlc-merge {storyKey}` — checks Jira status is Done, then merges story → sprint
- [ ] PR description includes Jira link and acceptance criteria
- [ ] Comment added to Jira story

## Next Step
After completion, suggest: `/sdlc-test {storyKey}`
