---
name: sdlc-stories
description: "SDLC Phase 2: Break a PRD into user stories with acceptance criteria"
---

# sdlc-stories — SDLC Phase 2: Requirements

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
- `.claude/SDLCs/prds/` — PRD documents
- `.claude/SDLCs/epics/` — epic documents for business context
- `CLAUDE.md` (if exists) — project conventions and tech stack

## Input
$ARGUMENTS — Either:
- A path to a PRD file (e.g., `prd-PROJ-002.md` or `.claude/SDLCs/prds/prd-PROJ-002.md`)
- A Jira Epic key (e.g., `PROJ-002`)

## Task
Decompose a PRD into Jira User Stories with story points estimate. Each story maps to one functional requirement or a coherent user-facing feature slice. Bulk create all stories in Jira under the Epic.

### Step 1: Load PRD Content
If $ARGUMENTS is a file path, read the PRD from `.claude/SDLCs/prds/`.
If $ARGUMENTS is a Jira key, read the Epic and its description, then fetch the local PRD file if it exists:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

### Step 2: Decompose into Stories
Apply INVEST criteria (Independent, Negotiable, Valuable, Estimable, Small, Testable):
- Each story should be completable in 1–3 days
- Format: "As a [role], I want [capability] so that [benefit]"
- Estimate story points using Fibonacci sequence: 1, 2, 3, 5, 8, 13
- Assign priority: Critical / High / Medium / Low
- Identify dependencies between stories

Produce 5–15 stories from the PRD functional requirements.

### Step 3: Write Story Files
Create one file per story in `.claude/SDLCs/stories/story-[EPIC-KEY]-[N].md`:

```markdown
# Story: [Story Title]

## Details
- Story Key: [PROJ-XXX — fill after creation]
- Epic Key: [EPIC-KEY]
- Priority: [Critical/High/Medium/Low]
- Story Points: [1/2/3/5/8/13]
- Status: To Do
- Type: [dev | ux | research | qa]
- Branch: [story/{KEY}-{title-slug} — filled when sprint assigned]
- Sprint Branch: [sprint/{N}-{slug} — filled when sprint assigned]

## User Story
As a [role],
I want [capability],
So that [benefit].

## Acceptance Criteria
- Given [context], When [action], Then [outcome]
- Given [context], When [action], Then [outcome]

## Technical Notes
[Any implementation hints or constraints relevant to this story]

## Dependencies
- Depends on: [story key or system]
- Blocks: [story key or system]
```

### Step 4: Update PRD Story Summary Table
Update `.claude/SDLCs/prds/prd-[EPIC-KEY].md` User Stories Summary table with all generated story titles and point estimates.

### Step 5: Jira Sync — Bulk Create Stories
Construct the bulk create payload with all stories:

```
# MCP: jira_create_issue (repeat per sub-task)
# fields: project.key, summary, issuetype, parent.key, description, labels
```

After bulk creation, update each story file with the assigned Jira key from the response.

Link dependent stories:

```
# MCP: jira_create_issue
# fields: project.key, summary, issuetype, epicLink, priority, description, labels
```

## Output
1. **Local files:** `.claude/SDLCs/stories/story-[EPIC-KEY]-[N].md` (one per story)
2. **Jira action:** Bulk create Stories under Epic; link dependencies
3. **Summary:** Print story count, total story points, and table of created stories with Jira keys

## Validation
Before finishing, confirm:
- [ ] All story files written to `.claude/SDLCs/stories/`
- [ ] Each story has INVEST-compliant user story and acceptance criteria
- [ ] Story points assigned to all stories
- [ ] Jira bulk create payload provided / executed
- [ ] PRD story summary table updated
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-acceptance [FIRST-STORY-KEY]`
