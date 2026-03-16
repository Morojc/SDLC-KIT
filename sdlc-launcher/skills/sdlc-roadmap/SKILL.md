---
name: sdlc-roadmap
description: "SDLC Phase 1: Generate a product roadmap from an epic"
---

# sdlc-roadmap — SDLC Phase 1: Planning

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
- `.claude/SDLCs/epics/` — epic documents
- `.claude/SDLCs/ideas/` — idea documents for context
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — Two values:
1. Epic key or epic file path (e.g., `PROJ-002` or `epic-001.md`)
2. Number of quarters to plan (e.g., `4` for a 4-quarter roadmap; defaults to 4)

Example: `/sdlc-roadmap PROJ-002 4`

## Task
Generate a quarterly roadmap with milestones, dependencies, and risks. Sync the roadmap to Jira as a version/fix-version timeline and add a roadmap comment to the Epic.

### Step 1: Load Epic Context
If $ARGUMENTS contains a Jira key, fetch the epic:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

If it's a file path, read the epic document from `.claude/SDLCs/epics/`.

### Step 2: Generate Quarterly Milestones
Based on the epic scope and today's date, produce a realistic quarter-by-quarter plan:

- **Q[N] — [Theme]**: Focus area, key deliverables, exit criteria
- Identify which stories/features belong to each quarter
- Mark dependencies between quarters
- Highlight risks per quarter with mitigation strategies

Use today's date to calculate actual quarter labels (e.g., Q2 2026, Q3 2026).

### Step 3: Write Roadmap Document
Create `.claude/SDLCs/epics/roadmap-[EPIC-KEY].md`:

```markdown
# Roadmap: [Epic Title]

## Epic Reference
- Epic Key: [PROJ-XXX]
- Planning Horizon: [Q1 YYYY – QN YYYY]
- Created: [date]
- Owner: [from jira-config if set]

## Quarterly Plan

### Q[N] [YYYY] — [Theme: e.g., "Foundation"]
**Goal:** [What success looks like at end of quarter]
**Deliverables:**
- [ ] [Milestone 1]
- [ ] [Milestone 2]
**Exit Criteria:** [How do we know Q[N] is done?]
**Dependencies:** [External teams, systems, or decisions needed]
**Risks:**
- Risk: [description] | Mitigation: [action]

### Q[N+1] [YYYY] — [Theme: e.g., "Core Features"]
...

## Dependency Map
```
Q[N] → Q[N+1]: [What must complete first]
Q[N+1] → Q[N+2]: [What must complete first]
```

## Risk Register
| Quarter | Risk | Likelihood | Impact | Mitigation |
|---------|------|------------|--------|------------|
| Q[N] | [description] | Medium | High | [action] |

## Open Questions
- [ ] [Decision needed by when]
```

### Step 4: Jira Sync — Add Roadmap as Epic Comment
Post the roadmap summary to the Jira Epic as a comment:

```
# MCP: jira_add_comment
# Post structured summary comment to the issue
```

Optionally create Jira Versions for each quarter milestone:

```
# MCP: use appropriate MCP tool instead of curl
```

## Output
1. **Local file:** `.claude/SDLCs/epics/roadmap-[EPIC-KEY].md`
2. **Jira action:** Add roadmap comment to Epic; optionally create Versions per quarter
3. **Summary:** Print quarters planned, milestone count, and key risks identified

## Validation
Before finishing, confirm:
- [ ] Roadmap file written to correct path
- [ ] All quarters have themes, deliverables, and exit criteria
- [ ] Dependency map and risk register populated
- [ ] Roadmap comment posted to Jira Epic
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-prd [EPIC-KEY]`
