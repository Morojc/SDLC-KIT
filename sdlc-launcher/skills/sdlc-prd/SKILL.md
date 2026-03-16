---
name: sdlc-prd
description: "SDLC Phase 2: Create a Product Requirements Document from an epic"
---

# sdlc-prd — SDLC Phase 2: Requirements

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
- `.claude/SDLCs/epics/` — epic documents and roadmaps
- `.claude/SDLCs/ideas/` — original idea for context
- `CLAUDE.md` (if exists) — project conventions and tech stack

## Input
$ARGUMENTS — Either:
- A Jira Epic key (e.g., `PROJ-002`)
- A path to an idea or epic file (e.g., `epic-001.md`)

## Task
Generate a full Product Requirements Document (PRD) from the Epic. The PRD defines functional requirements, non-functional requirements, constraints, and user stories summary. Attach it to the Jira Epic.

### Step 1: Load Epic Context
If $ARGUMENTS is a Jira key, fetch the epic details:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

Also fetch any linked issues to understand existing context:

```
# MCP: jira_search_issues
# JQL query to find relevant issues
```

If it's a file path, read the epic/idea document directly.

### Step 2: Draft the PRD
Produce a comprehensive PRD with:
- Executive summary (2-3 sentences)
- Problem and opportunity with user impact
- Goals (what success looks like) and non-goals (explicit exclusions)
- Functional requirements (FR-001, FR-002, ...) with Given/When/Then acceptance criteria
- Non-functional requirements: performance, security, scalability, accessibility
- Technical constraints from CLAUDE.md or project context
- User story summary table (placeholder keys if stories not yet created)
- Timeline aligned to roadmap quarters
- Open questions that need answers before development starts

### Step 3: Write PRD Document
Create `.claude/SDLCs/prds/prd-[EPIC-KEY].md`:

```markdown
# Product Requirements Document
## Epic: [EPIC-KEY] — [Epic Title]

## Overview
[Executive summary — what is being built and why, in 2-3 sentences]

## Problem & Opportunity
[Problem definition with measurable user impact. Include data or user research if available.]

## Goals & Non-Goals

### Goals
- [Goal 1 — measurable]
- [Goal 2 — measurable]

### Non-Goals
- [Explicit exclusion 1]
- [Explicit exclusion 2]

## User Stories Summary
| Story Key | Title | Priority | Points |
|-----------|-------|----------|--------|
| TBD | [As a user, I want...] | High | 5 |

## Functional Requirements

### FR-001: [Requirement Name]
**Description:** [What the system must do]
**Acceptance Criteria:**
- Given [context], When [action], Then [outcome]
- Given [context], When [action], Then [outcome]

### FR-002: [Requirement Name]
...

## Non-Functional Requirements
| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | API response time | < 200ms p95 |
| Security | Authentication | OAuth 2.0 / API token |
| Scalability | Concurrent users | 1,000+ |
| Availability | Uptime | 99.9% |
| Accessibility | WCAG compliance | Level AA |

## Technical Constraints
[Stack, existing dependencies, legacy system integrations, team expertise limits]

## Out of Scope
[Items explicitly excluded from this epic/release]

## Timeline
| Phase | Duration | Milestone |
|-------|----------|-----------|
| Discovery | 1 week | Requirements signed off |
| Design | 1 week | Tech design approved |
| Development | [N] weeks | Feature complete |
| Testing | 1 week | QA passed |
| Release | 1 day | Deployed to production |

## Open Questions
- [ ] [Question 1 — owner and due date]
- [ ] [Question 2 — owner and due date]

## Jira Reference
- Epic Key: [EPIC-KEY]
- PRD Created: [date]
- Status: Draft
```

### Step 4: Jira Sync — Attach PRD to Epic
Update the Epic description with a summary and add a comment with the PRD:

```
# MCP: jira_get_issue  or  jira_update_issue
# Get or update the target issue
```

## Output
1. **Local file:** `.claude/SDLCs/prds/prd-[EPIC-KEY].md`
2. **Jira action:** Update Epic description with PRD summary; add PRD comment
3. **Summary:** Print FR count, NFR count, open question count

## Validation
Before finishing, confirm:
- [ ] PRD file written to `.claude/SDLCs/prds/prd-[EPIC-KEY].md`
- [ ] At least 3 functional requirements with Gherkin criteria
- [ ] NFR table covers performance, security, scalability
- [ ] Jira Epic updated with PRD summary
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-stories prd-[EPIC-KEY].md`
