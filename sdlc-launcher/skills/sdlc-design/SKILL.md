---
name: sdlc-design
description: "SDLC Phase 3: Create a technical design document for a story or feature"
---

# sdlc-design — SDLC Phase 3: Design

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
- `.claude/SDLCs/stories/` — story and acceptance criteria documents
- `.claude/SDLCs/prds/` — PRD for functional and non-functional requirements
- `.claude/SDLCs/epics/` — epic for business context
- `CLAUDE.md` (if exists) — project conventions, tech stack, coding standards

## Input
$ARGUMENTS — Either:
- A Jira Story key (e.g., `PROJ-010`)
- A feature name or description (e.g., `"real-time document sync"`)

## Task
Create a technical design document covering data models, API contracts, UI flows, and implementation approach. This serves as the blueprint for development. Add the design as a comment on the Jira Story.

### Step 1: Load Context
If $ARGUMENTS is a Jira key, fetch the story details and acceptance criteria:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

Read related story files from `.claude/SDLCs/stories/`.
Read `CLAUDE.md` for tech stack constraints (language, frameworks, patterns).

### Step 2: Produce Technical Design
Cover all applicable sections:

**Data Models:** Define new or modified entities with field types, constraints, and relationships.

**API Contracts:** For each endpoint — method, path, request/response schemas, error codes, authentication requirements.

**UI/UX Flows:** If user-facing — user journey steps, component interactions, state transitions.

**Service Architecture:** How this feature fits into existing services. New services needed? Event-driven patterns?

**State Management:** How data flows through the system. Caching strategy. Consistency requirements.

**Security Design:** Auth/authz for new endpoints. Data validation. Sensitive data handling.

**Migration Plan:** Database migrations needed. Backward compatibility. Feature flags.

### Step 3: Write Design Document
Create `.claude/SDLCs/designs/design-[STORY-KEY or feature-slug].md`:

```markdown
# Technical Design: [Feature/Story Title]

## Design Reference
- Story Key: [PROJ-XXX]
- Epic Key: [PROJ-YYY]
- Created: [date]
- Status: Draft
- Reviewer: [TBD]

## Overview
[2-3 sentence technical summary of what is being built and how]

## Data Models

### [Model Name]
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, NOT NULL | Unique identifier |
| [field] | [type] | [constraints] | [description] |

**Relationships:**
- [Model] has many [OtherModel] via [foreign_key]

## API Contracts

### [METHOD] /api/v1/[endpoint]
**Authentication:** Bearer token / API key / Public
**Request:**
```json
{
  "[field]": "[type — description]"
}
```
**Response (200):**
```json
{
  "[field]": "[type — description]"
}
```
**Error Responses:**
| Code | Reason |
|------|--------|
| 400 | Invalid input |
| 401 | Unauthorized |
| 404 | Resource not found |
| 500 | Internal server error |

## UI/UX Flow
[If applicable — describe the user journey step by step]

1. User [action] on [screen/component]
2. System [response]
3. User sees [feedback]

## Service Architecture
[Diagram description or mermaid diagram]

```
[Component A] → [API Gateway] → [Service B] → [Database]
                                     ↓
                              [Event Queue] → [Service C]
```

## Security Considerations
- [Authentication requirement]
- [Authorization rule — who can do what]
- [Data validation rule]
- [Sensitive data handling]

## Migration Plan
- [ ] Database migration: [description]
- [ ] Feature flag: `[flag-name]` — roll out to [%] initially
- [ ] Backward compatibility: [how existing clients are handled]

## Open Design Questions
- [ ] [Decision needed — options and trade-offs]
- [ ] [Performance concern to validate]

## Alternatives Considered
| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| [Option A] | [pros] | [cons] | Chosen / Rejected |
| [Option B] | [pros] | [cons] | Chosen / Rejected |
```

### Step 4: Jira Sync — Add Design as Comment
Post the design document summary to the Jira Story:

```
# MCP: jira_add_comment
# Post structured summary comment to the issue
```

## Output
1. **Local file:** `.claude/SDLCs/designs/design-[KEY].md`
2. **Jira action:** Add design summary comment to Story
3. **Summary:** Print model count, API endpoint count, open questions

## Validation
Before finishing, confirm:
- [ ] Design file written to `.claude/SDLCs/designs/`
- [ ] Data models, API contracts, and security sections populated
- [ ] Migration plan documented
- [ ] Alternatives considered section filled
- [ ] Jira comment added to Story
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-arch design-[KEY].md`
