---
name: sdlc-merge
description: "SDLC Phase 4: Merge a story branch into its sprint, or a sprint branch into main — guarded by Jira status checks"
---

# sdlc-merge — Guarded Branch Merge

## MCP Tools
- **Jira** (`jira` MCP): `jira_get_issue`, `jira_search_issues`, `jira_transition_issue` — status checks and transitions
- **GitHub** (`github` MCP): `create_pull_request`, `get_pull_request`, `merge_pull_request`, `list_pull_requests`

## Mulch & Seeds

### Session Start
```bash
mulch prime --context
sd prime
```

### Session Close
```bash
mulch learn
mulch record sdlc-commands --type convention --description "..."
mulch sync
sd close <id>
sd sync
```

## Context Loading
- `.claude/settings/jira-config.json` — project key, board ID
- `.claude/SDLCs/stories/` — story files (sprint assignment, story type)
- `CLAUDE.md` — branch naming conventions

## Input
$ARGUMENTS — One of:
- A story key: `MA-3` → merge story branch into its sprint branch
- A sprint ref: `sprint/1` or `sprint-1` → merge sprint branch into `main`

---

## Branch Naming Reference

| Level | Pattern | Example |
|-------|---------|---------|
| Sprint branch | `sprint/{N}-{slug}` | `sprint/1-foundation` |
| Story branch | `story/{KEY}-{slug}` | `story/MA-3-create-task` |
| Release target | `main` | `main` |

---

## Task

### Detect Merge Level

If $ARGUMENTS is a **Jira story key** (e.g. `MA-3`) → run **Story → Sprint merge**.
If $ARGUMENTS is a **sprint ref** (e.g. `sprint/1`) → run **Sprint → Main merge**.

---

## LEVEL 1: Story → Sprint Merge

### Step 1: Check Jira Status

```
# MCP: jira_get_issue
# Fetch: status, summary, sprint, assignee for {storyKey}
```

**Status gate:**

```
IF status == "Done"
  → proceed to Step 2
ELSE
  → BLOCK merge. Print:

  ✗ MERGE BLOCKED — MA-3 is not Done
  ──────────────────────────────────────
  Current status : In Progress
  Required status: Done
  ──────────────────────────────────────
  Complete the implementation first:
    /sdlc-implement MA-3
  Then mark the story Done in Jira before merging.

  → EXIT
```

### Step 2: Resolve Branch Names

From the story file `.claude/SDLCs/stories/story-{KEY}.md`, read:
- `sprintBranch` field (e.g. `sprint/1-foundation`)
- `branch` field (e.g. `story/MA-3-create-task`)

If fields are missing, derive them:
- Story branch: `story/{KEY}-{title-slug}`
- Sprint branch: look up open sprint in Jira → `sprint/{N}-{goal-slug}`

### Step 3: Check for Conflicts

```
# MCP: github — get_pull_request or list_pull_requests
# Check if a PR already exists for this story → sprint
```

If PR already exists and is open → print its URL, ask user to review and merge directly.
If no PR exists → proceed to Step 4.

### Step 4: Open Pull Request

```
# MCP: github — create_pull_request
# title:  "story/MA-3: Create a financial task"
# head:   "story/MA-3-create-task"
# base:   "sprint/1-foundation"
# body:
  ## Story: MA-3 — Create a Financial Task
  **Jira:** {JIRA_BASE_URL}/browse/MA-3
  **Sprint:** sprint/1-foundation
  **Status at merge:** Done ✓

  ### Changes
  {list key files changed}

  ### Acceptance Criteria
  {paste ACs from story file}
```

### Step 5: Transition Jira Story

```
# MCP: jira_transition_issue
# Transition MA-3 → "In Review" (if not already Done+merged)
```

### Step 6: Print Result

```
✓ PR OPENED — story/MA-3-create-task → sprint/1-foundation
──────────────────────────────────────────────────────────
PR URL   : {url}
Status   : Open — awaiting review
Jira     : MA-3 transitioned to In Review
──────────────────────────────────────────────────────────
Once merged, run: /sdlc-merge sprint/1  (when all stories are done)
```

---

## LEVEL 2: Sprint → Main Merge

### Step 1: Identify Sprint and Its Stories

```
# MCP: jira_search_issues
# JQL: project={PROJECT_KEY} AND sprint="{sprintName}" AND issuetype=Story
# Fetch: key, summary, status for every story in the sprint
```

### Step 2: Check ALL Stories Are Done

Build a status table:

```
SPRINT MERGE CHECK — sprint/1-foundation → main
──────────────────────────────────────────────────
  MA-3   Create a financial task       Done  ✓
  MA-5   Complete task + timestamp     Done  ✓
  MA-10  Data persistence              Done  ✓
  MA-12  Authentication                In Progress  ✗
──────────────────────────────────────────────────
Result: BLOCKED — 1 story not Done
```

**If ANY story is not Done → BLOCK:**

```
✗ SPRINT MERGE BLOCKED
──────────────────────────────────────────────────
The following stories must be Done before sprint/1
can merge into main:

  MA-12  Authentication  [In Progress]
    → Complete implementation: /sdlc-implement MA-12
    → Merge story branch:      /sdlc-merge MA-12

──────────────────────────────────────────────────
Re-run /sdlc-merge sprint/1 when all stories are Done.
→ EXIT
```

**If ALL stories are Done → proceed.**

### Step 3: Check All Story Branches Are Merged

```
# MCP: github — list_pull_requests
# base: sprint/1-foundation
# state: open
```

If any story PRs are still open → list them and block:

```
✗ SPRINT MERGE BLOCKED — open story PRs remain:
  story/MA-12-authentication  [open PR #14]
  Merge or close these PRs first.
→ EXIT
```

### Step 4: Open Sprint → Main Pull Request

```
# MCP: github — create_pull_request
# title:  "sprint/1-foundation → main"
# head:   "sprint/1-foundation"
# base:   "main"
# body:
  ## Sprint 1 — Foundation
  **Stories merged:** {N}
  **Total points:** {X}

  ### Stories
  | Key | Title | Points |
  |-----|-------|--------|
  | MA-3  | Create a financial task   | 3 |
  | MA-5  | Complete task + timestamp | 2 |
  | MA-10 | Data persistence          | 5 |
  | MA-12 | Authentication            | 5 |

  ### Definition of Done
  - [x] All story branches merged to sprint
  - [x] All Jira stories status = Done
  - [x] CI passing on sprint branch
```

### Step 5: Close Sprint in Jira

```
# MCP: jira_transition_issue (or close sprint via board API)
# Close Sprint 1 in Jira
```

### Step 6: Print Result

```
✓ PR OPENED — sprint/1-foundation → main
──────────────────────────────────────────────────
PR URL   : {url}
Stories  : 4 merged | 15 pts
Sprint   : Closed in Jira
──────────────────────────────────────────────────
Once merged, run: /sdlc-release  to tag the version
```

---

## Output
- PR URL (story → sprint, or sprint → main)
- Jira status transitions logged
- Blocked merge: clear error with the exact blocker and fix command

## Validation
- [ ] Jira status checked before any merge action
- [ ] Merge blocked if status ≠ Done (story level) or any story ≠ Done (sprint level)
- [ ] PR created with correct head/base branches
- [ ] Jira transitions logged
- [ ] Sprint closed when sprint merges to main

## Next Step
- After story merge: `/sdlc-merge sprint/{N}` when all stories are done
- After sprint merge: `/sdlc-release` to tag the release
