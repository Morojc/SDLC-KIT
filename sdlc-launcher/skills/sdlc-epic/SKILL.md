---
name: sdlc-epic
description: "SDLC Phase 1: Create a Jira epic from an idea document"
---

# sdlc-epic — SDLC Phase 1: Planning

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
- `.claude/SDLCs/ideas/` — idea documents
- `.claude/SDLCs/epics/` — existing epics for numbering continuity
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — Either:
- A path to an idea file (e.g., `idea-001.md` or `.claude/SDLCs/ideas/idea-001.md`)
- A Jira Initiative key (e.g., `PROJ-001`)

## Task
Convert an idea into a full Jira Epic with description, business value, and acceptance criteria. Write the epic artifact locally and create the Jira Epic ticket linked to the Initiative.

### Step 1: Load Source Material
If $ARGUMENTS is a file path, read the idea document from `.claude/SDLCs/ideas/`.
If $ARGUMENTS is a Jira key, provide a curl to fetch the issue:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

### Step 2: Expand into Epic
Using the idea content, produce:
- A clear epic title (action-oriented, e.g., "Implement Real-Time Document Collaboration")
- Business value statement (why this matters to the organization)
- Scope definition (what's in and explicitly what's out)
- High-level acceptance criteria (3–5 measurable conditions)
- Estimated complexity (S/M/L/XL with rationale)
- Dependencies on other systems or teams

### Step 3: Write Epic Document
Create `.claude/SDLCs/epics/epic-XXX.md`:

```markdown
# Epic: [Epic Title]

## Jira Reference
- Epic Key: [PROJ-XXX — fill after creation]
- Initiative Key: [Parent initiative key]
- Created: [date]
- Status: To Do
- Estimated Size: [S/M/L/XL]

## Business Value
[Why does this matter? What business outcome does it drive?]

## Scope

### In Scope
- [Item 1]
- [Item 2]

### Out of Scope
- [Item 1]
- [Item 2]

## Acceptance Criteria
- [ ] [Measurable condition 1]
- [ ] [Measurable condition 2]
- [ ] [Measurable condition 3]

## Dependencies
- [System or team dependency 1]
- [System or team dependency 2]

## Stories Overview
[To be populated by /sdlc-stories]

## Notes
[Any additional context or open questions]
```

### Step 4: Jira Sync — Create Epic
Create the Epic in Jira and link it to the Initiative.

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

Update the epic file's `Jira Reference` with the returned epic key.

## Output
1. **Local file:** `.claude/SDLCs/epics/epic-XXX.md`
2. **Jira action:** Create Epic linked to Initiative; set epic name custom field
3. **Summary:** Print epic title, file path, and Jira key

## Validation
Before finishing, confirm:
- [ ] Epic file written to `.claude/SDLCs/epics/epic-XXX.md`
- [ ] Business value and scope clearly defined
- [ ] Acceptance criteria are measurable (not vague)
- [ ] Jira Epic created and linked to Initiative
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-roadmap [EPIC-KEY] 4`
