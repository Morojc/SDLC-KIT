---
name: sdlc-acceptance
description: "SDLC Phase 2: Generate acceptance criteria for user stories"
---

# sdlc-acceptance — SDLC Phase 2: Requirements

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
- `.claude/SDLCs/stories/` — story documents
- `.claude/SDLCs/prds/` — PRD for functional requirement context
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — A Jira Story key (e.g., `PROJ-010`)

## Task
Write comprehensive Gherkin-style acceptance criteria for a Jira Story. Update the story's local file and push the criteria to the Jira issue's acceptance criteria custom field and description.

### Step 1: Load Story Details
Fetch the story from Jira:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

Also read the local story file if it exists in `.claude/SDLCs/stories/`.

Fetch the parent Epic for additional context:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

### Step 2: Write Acceptance Criteria
For each scenario, use the Gherkin format:

```
Scenario: [Descriptive name]
  Given [initial context / precondition]
  When [action taken by user or system]
  Then [expected outcome]
  And [additional outcome if needed]
```

Cover:
- **Happy path**: The expected successful flow
- **Edge cases**: Boundary conditions, empty states, maximum values
- **Error cases**: Invalid input, missing data, system failures
- **Security**: Unauthorized access attempts, permission boundaries
- **Performance**: If response time is critical, include timing criteria

Aim for 5–10 scenarios per story. Each must be independently verifiable.

### Step 3: Update Local Story File
Update `.claude/SDLCs/stories/story-[STORY-KEY].md` (or the matching story file) with the full acceptance criteria:

```markdown
## Acceptance Criteria (Gherkin)

### Scenario 1: [Happy path — primary flow]
```gherkin
Given [context]
When [action]
Then [outcome]
```

### Scenario 2: [Edge case]
```gherkin
Given [context]
When [edge condition]
Then [expected behavior]
```

### Scenario 3: [Error case]
```gherkin
Given [context]
When [invalid action]
Then [error message or fallback behavior]
```

### Scenario 4: [Permission / security]
```gherkin
Given [user without permission]
When [they attempt action]
Then [access denied with appropriate message]
```

## Definition of Done
- [ ] All Gherkin scenarios have passing automated tests
- [ ] Code reviewed and approved
- [ ] No regression in related features
- [ ] Documentation updated if user-facing
- [ ] Jira ticket transitioned to Done
```

### Step 4: Jira Sync — Set Acceptance Criteria Field
Update the Jira story with the acceptance criteria in the custom field and add a comment:

```
# MCP: jira_get_issue  or  jira_update_issue
# Get or update the target issue
```

## Output
1. **Local file:** Updated `.claude/SDLCs/stories/story-[STORY-KEY].md` with Gherkin criteria
2. **Jira action:** Set `customfield_10016` (acceptance criteria); update story description; add comment
3. **Summary:** Print scenario count and categories covered

## Validation
Before finishing, confirm:
- [ ] Story file updated with complete Gherkin scenarios
- [ ] Happy path, edge cases, and error cases all covered
- [ ] Jira `customfield_10016` set with acceptance criteria
- [ ] Story description updated in Jira
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-design $ARGUMENTS`
