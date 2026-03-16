# SPEC.md — Claude SDLC Plugin with Jira Integration

> **Purpose:** This specification defines a Claude Code plugin that manages the complete Software Development Lifecycle (SDLC) — from raw idea to deployed product — using structured slash commands, AI-driven artifact generation, and deep Jira integration.
>
> **Inspired by:** [Atlassian Agile SDLC](https://www.atlassian.com/agile/software-development/sdlc) · [PRPs-agentic-eng](https://github.com/Wirasm/PRPs-agentic-eng)

---

## Table of Contents

1. [Vision & Goals](#1-vision--goals)
2. [SDLC Phases Overview](#2-sdlc-phases-overview)
3. [Plugin Architecture](#3-plugin-architecture)
4. [Command Reference](#4-command-reference)
5. [Jira Integration Layer](#5-jira-integration-layer)
6. [Full Workflow: Idea → Production](#6-full-workflow-idea--production)
7. [Artifact Schema](#7-artifact-schema)
8. [Prompt Template Structure](#8-prompt-template-structure)
9. [Jira API Mapping](#9-jira-api-mapping)
10. [Configuration & Setup](#10-configuration--setup)
11. [Directory Structure](#11-directory-structure)

---

## 1. Vision & Goals

### What This Plugin Does

This plugin transforms Claude Code into a **full SDLC co-pilot** — capable of taking a single idea and orchestrating it through every phase of software development while keeping Jira in perfect sync with each step.

### Core Goals

| Goal | Description |
|------|-------------|
| **Idea → Ticket** | Convert raw ideas into structured Jira Epics, Stories, and Tasks automatically |
| **Ticket → Plan** | Generate implementation plans from Jira issues |
| **Plan → Code** | Execute AI-driven development with validation loops |
| **Code → Review** | Automated code review tied back to Jira |
| **Review → Deploy** | Deployment checklists synced with Jira release tracking |
| **Deploy → Monitor** | Post-deployment monitoring and Jira ticket closure |

### Design Principles

1. **Every action produces a Jira artifact** — no SDLC step is undocumented
2. **Commands are composable** — each command can run standalone or as part of a pipeline
3. **AI-first, human-validated** — Claude generates, humans approve critical actions
4. **Context-aware** — commands read project context, Jira state, and codebase before acting
5. **Traceability** — every commit, PR, and code change links back to a Jira issue

---

## 2. SDLC Phases Overview

Based on the Atlassian Agile SDLC model, the plugin covers **7 phases**, each mapped to specific commands and Jira artifacts:

```
IDEA
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 1: PLANNING                                       │
│  /sdlc-idea → /sdlc-epic → /sdlc-roadmap               │
│  Jira: Epic + Initiative + Roadmap                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 2: REQUIREMENTS                                   │
│  /sdlc-prd → /sdlc-stories → /sdlc-acceptance          │
│  Jira: Stories + Acceptance Criteria + Sprint Backlog   │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 3: DESIGN                                         │
│  /sdlc-design → /sdlc-arch                             │
│  Jira: Design Tasks + Architecture Decision Records      │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 4: DEVELOPMENT                                    │
│  /sdlc-plan → /sdlc-implement → /sdlc-commit           │
│  Jira: Sub-tasks + In Progress + Time Tracking          │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 5: TESTING                                        │
│  /sdlc-test → /sdlc-bug → /sdlc-qa                    │
│  Jira: Bug Reports + Test Coverage + QA Checklist       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 6: DEPLOYMENT                                     │
│  /sdlc-release → /sdlc-deploy                          │
│  Jira: Release Ticket + Deployment Checklist            │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 7: MAINTENANCE                                    │
│  /sdlc-retro → /sdlc-monitor → /sdlc-hotfix           │
│  Jira: Retrospective + Bug Tracking + Hotfix Flow       │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Plugin Architecture

### Folder Structure

```
.claude/
├── commands/
│   └── sdlc/
│       ├── phase-1-planning/
│       │   ├── sdlc-idea.md
│       │   ├── sdlc-epic.md
│       │   └── sdlc-roadmap.md
│       ├── phase-2-requirements/
│       │   ├── sdlc-prd.md
│       │   ├── sdlc-stories.md
│       │   └── sdlc-acceptance.md
│       ├── phase-3-design/
│       │   ├── sdlc-design.md
│       │   └── sdlc-arch.md
│       ├── phase-4-development/
│       │   ├── sdlc-plan.md
│       │   ├── sdlc-implement.md
│       │   └── sdlc-commit.md
│       ├── phase-5-testing/
│       │   ├── sdlc-test.md
│       │   ├── sdlc-bug.md
│       │   └── sdlc-qa.md
│       ├── phase-6-deployment/
│       │   ├── sdlc-release.md
│       │   └── sdlc-deploy.md
│       ├── phase-7-maintenance/
│       │   ├── sdlc-retro.md
│       │   ├── sdlc-monitor.md
│       │   └── sdlc-hotfix.md
│       └── utils/
│           ├── sdlc-status.md
│           ├── sdlc-sync-jira.md
│           └── sdlc-sprint.md
├── agents/
│   ├── jira-connector.md
│   ├── requirements-analyst.md
│   ├── tech-architect.md
│   ├── qa-engineer.md
│   └── release-manager.md
├── SDLCs/
│   ├── ideas/
│   ├── prds/
│   ├── epics/
│   ├── plans/
│   │   └── completed/
│   ├── designs/
│   ├── test-reports/
│   ├── releases/
│   └── retros/
└── settings/
    └── jira-config.json
```

---

## 4. Command Reference

### Phase 1 — Planning

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-idea` | `"<idea description>"` | Refines a raw idea into a structured concept with problem statement, target users, and success metrics. Creates a Jira Initiative. |
| `/sdlc-epic` | `[idea-file or JIRA-KEY]` | Converts an idea into a full Jira Epic with description, business value, and acceptance criteria. |
| `/sdlc-roadmap` | `[epic-key] [quarters]` | Generates a roadmap with quarterly milestones, dependencies, and risks. Syncs to Jira. |

### Phase 2 — Requirements

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-prd` | `[epic-key or idea-file]` | Generates a full Product Requirements Document from an Epic. |
| `/sdlc-stories` | `[prd-file or epic-key]` | Decomposes a PRD into Jira User Stories with story points estimate. |
| `/sdlc-acceptance` | `[story-key]` | Writes Gherkin-style acceptance criteria for a Jira Story. |

### Phase 3 — Design

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-design` | `[story-key or feature-name]` | Creates a technical design document: data models, API contracts, UI flows. |
| `/sdlc-arch` | `[design-file or epic-key]` | Produces an Architecture Decision Record (ADR) with trade-off analysis. |

### Phase 4 — Development

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-plan` | `[story-key or design-file]` | Creates a step-by-step implementation plan with file paths, dependencies, and validation commands. Creates Jira Sub-tasks. |
| `/sdlc-implement` | `[plan-file or story-key]` | Executes the implementation plan with iterative validation loops. Transitions Jira ticket to In Progress. |
| `/sdlc-commit` | `[story-key]` | Generates a conventional commit message linked to the Jira ticket and creates a PR description. |

### Phase 5 — Testing

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-test` | `[story-key or file-path]` | Generates a test plan and writes unit/integration/e2e tests. |
| `/sdlc-bug` | `"<bug description>"` | Creates a Jira Bug ticket with reproduction steps, severity, and affected version. |
| `/sdlc-qa` | `[story-key or release-key]` | Runs a full QA checklist against acceptance criteria. Transitions ticket to Review. |

### Phase 6 — Deployment

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-release` | `[version] [epic-key]` | Creates a Jira Release, generates release notes from completed stories, and produces a deployment checklist. |
| `/sdlc-deploy` | `[release-key] [environment]` | Executes deployment validation, updates Jira tickets to Done, and notifies stakeholders. |

### Phase 7 — Maintenance

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-retro` | `[sprint-id or release-key]` | Generates a sprint/release retrospective with what went well, improvements, and action items as Jira Tasks. |
| `/sdlc-monitor` | `[release-key]` | Post-deployment monitoring checklist — error rates, performance, user feedback. Creates Jira tickets for issues found. |
| `/sdlc-hotfix` | `[bug-key]` | Initiates an expedited hotfix workflow: branch strategy, expedited review, and emergency deploy checklist. |

### Utility Commands

| Command | Arguments | Description |
|---------|-----------|-------------|
| `/sdlc-status` | `[epic-key or sprint-id]` | Displays current SDLC phase, open tickets, blockers, and next recommended action. |
| `/sdlc-sync-jira` | `[artifact-file]` | Pushes a locally generated artifact (PRD, plan, design) to the corresponding Jira issue as a description or comment. |
| `/sdlc-sprint` | `[epic-key] [sprint-number]` | Plans a sprint: assigns stories, estimates capacity, identifies risks, and creates the sprint in Jira. |

---

## 5. Jira Integration Layer

### Jira Atlassian MCP 

```json
// .claude/settings/jira-config.json
{
  "jira_base_url": "https://your-org.atlassian.net",
  "project_key": "PROJ",
  "auth_type": "api_token",
  "api_token_env": "JIRA_API_TOKEN",
  "email_env": "JIRA_EMAIL",
  "default_board_id": null,
  "default_sprint_id": null,
  "issue_types": {
    "initiative": "Initiative",
    "epic": "Epic",
    "story": "Story",
    "task": "Task",
    "sub_task": "Sub-task",
    "bug": "Bug"
  },
  "custom_fields": {
    "story_points": "story_points",
    "epic_link": "epic_link",
    "sprint": "sprint",
    "acceptance_criteria": "customfield_10016"
  },
  "workflow_transitions": {
    "to_do": "To Do",
    "in_progress": "In Progress",
    "in_review": "In Review",
    "done": "Done"
  }
}
```

### Jira Issue Type Hierarchy

```
Initiative (Portfolio level)
  └── Epic (Feature level)
        └── Story (User-facing requirement)
              ├── Sub-task (Implementation unit)
              └── Bug (Defect)
        └── Task (Technical/non-user-facing)
              └── Sub-task
```

### Jira API Endpoints Used

| Action | Method | Endpoint |
|--------|--------|----------|
| Create Issue | POST | `/rest/api/3/issue` |
| Update Issue | PUT | `/rest/api/3/issue/{issueKey}` |
| Transition Issue | POST | `/rest/api/3/issue/{issueKey}/transitions` |
| Add Comment | POST | `/rest/api/3/issue/{issueKey}/comment` |
| Get Epic Stories | GET | `/rest/agile/1.0/board/{boardId}/epic/{epicId}/issue` |
| Create Sprint | POST | `/rest/agile/1.0/sprint` |
| Move to Sprint | POST | `/rest/agile/1.0/sprint/{sprintId}/issue` |
| Get Board | GET | `/rest/agile/1.0/board/{boardId}` |
| Create Release | POST | `/rest/api/3/version` |
| Get Transitions | GET | `/rest/api/3/issue/{issueKey}/transitions` |
| Bulk Create Issues | POST | `/rest/api/3/issue/bulk` |
| Link Issues | POST | `/rest/api/3/issueLink` |
| Search Issues (JQL) | GET | `/rest/api/3/search?jql={query}` |

---

## 6. Full Workflow: Idea → Production

### Step-by-Step Journey

```
STEP 1 ─ Capture the Idea
  Command:  /sdlc-idea "Build a real-time collaboration feature for documents"
  Output:   .claude/SDLCs/ideas/idea-001.md
  Jira:     Creates Initiative: PROJ-001

STEP 2 ─ Define the Epic
  Command:  /sdlc-epic idea-001.md
  Output:   .claude/SDLCs/epics/epic-001.md
  Jira:     Creates Epic: PROJ-002, links to Initiative PROJ-001

STEP 3 ─ Write the PRD
  Command:  /sdlc-prd PROJ-002
  Output:   .claude/SDLCs/prds/prd-PROJ-002.md
  Jira:     Attaches PRD as Epic description + comment

STEP 4 ─ Break into Stories
  Command:  /sdlc-stories prd-PROJ-002.md
  Output:   .claude/SDLCs/stories/ (one file per story)
  Jira:     Bulk creates Stories under Epic PROJ-002
            (PROJ-010 through PROJ-018)

STEP 5 ─ Write Acceptance Criteria
  Command:  /sdlc-acceptance PROJ-010
  Output:   Updates story file + Jira custom field
  Jira:     Sets acceptance criteria on PROJ-010

STEP 6 ─ Technical Design
  Command:  /sdlc-design PROJ-010
  Output:   .claude/SDLCs/designs/design-PROJ-010.md
  Jira:     Adds design doc as comment on PROJ-010

STEP 7 ─ Sprint Planning
  Command:  /sdlc-sprint PROJ-002 1
  Output:   Sprint plan with capacity and risk analysis
  Jira:     Creates Sprint 1, assigns stories PROJ-010 to PROJ-014

STEP 8 ─ Implementation Plan
  Command:  /sdlc-plan PROJ-010
  Output:   .claude/SDLCs/plans/plan-PROJ-010.md
  Jira:     Creates Sub-tasks, transitions PROJ-010 → In Progress

STEP 9 ─ Implement
  Command:  /sdlc-implement plan-PROJ-010.md
  Output:   Working code with passing validation
  Jira:     Time logged on PROJ-010 sub-tasks

STEP 10 ─ Commit & PR
  Command:  /sdlc-commit PROJ-010
  Output:   Conventional commit + PR description
  Jira:     Commit message: "feat(PROJ-010): implement real-time sync"

STEP 11 ─ Testing
  Command:  /sdlc-test PROJ-010
  Output:   Test suite + coverage report
  Jira:     QA sub-task created under PROJ-010

STEP 12 ─ QA Validation
  Command:  /sdlc-qa PROJ-010
  Output:   QA checklist results
  Jira:     Transitions PROJ-010 → In Review

STEP 13 ─ Release
  Command:  /sdlc-release v1.2.0 PROJ-002
  Output:   .claude/SDLCs/releases/release-v1.2.0.md
  Jira:     Creates Version v1.2.0, generates release notes

STEP 14 ─ Deploy
  Command:  /sdlc-deploy v1.2.0 production
  Output:   Deployment validation report
  Jira:     All stories → Done, version released

STEP 15 ─ Retrospective
  Command:  /sdlc-retro PROJ-SPRINT-1
  Output:   .claude/SDLCs/retros/retro-sprint-1.md
  Jira:     Action items created as Tasks in next sprint

STEP 16 ─ Monitor
  Command:  /sdlc-monitor v1.2.0
  Output:   Monitoring checklist + alert summary
  Jira:     Any issues found → Bug tickets auto-created
```

---

## 7. Artifact Schema

### Idea Document (`idea-XXX.md`)

```markdown
# Idea: [Title]

## Problem Statement
[What problem does this solve?]

## Target Users
[Who benefits and how?]

## Proposed Solution
[High-level solution description]

## Success Metrics
- Metric 1: [measurable outcome]
- Metric 2: [measurable outcome]

## Risks & Assumptions
- Risk 1: [description]
- Assumption 1: [description]

## Jira Reference
- Initiative Key: [PROJ-XXX]
- Created: [date]
- Status: [To Do]
```

### PRD Document (`prd-PROJ-XXX.md`)

```markdown
# Product Requirements Document
## Epic: [PROJ-XXX] — [Epic Name]

## Overview
[Executive summary]

## Problem & Opportunity
[Problem definition with user impact]

## Goals & Non-Goals
### Goals
- Goal 1
### Non-Goals
- Non-goal 1

## User Stories Summary
| Story Key | Title | Priority | Points |
|-----------|-------|----------|--------|

## Functional Requirements
### FR-001: [Requirement Name]
**Description:** ...
**Acceptance Criteria:**
- Given / When / Then

## Non-Functional Requirements
| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | Response time | < 200ms |
| Security | Auth | OAuth 2.0 |

## Technical Constraints
[Stack, dependencies, legacy constraints]

## Out of Scope
[Explicitly excluded items]

## Timeline
| Phase | Duration | Milestone |
|-------|----------|-----------|

## Open Questions
- [ ] Question 1
```

### Implementation Plan (`plan-PROJ-XXX.md`)

```markdown
# Implementation Plan: [PROJ-XXX] — [Story Title]

## Jira Context
- Story Key: PROJ-XXX
- Epic: PROJ-YYY
- Sprint: Sprint N
- Assignee: [name]

## Implementation Phases
| Phase | Task | Status | Depends On | Parallel? |
|-------|------|--------|------------|-----------|
| 1 | Setup data models | pending | — | No |
| 2 | Build API layer | pending | Phase 1 | No |
| 3 | Write tests | pending | Phase 2 | Yes |
| 4 | Update UI | pending | Phase 2 | Yes |

## File Targets
- `src/models/[model].ts` — data schema
- `src/api/[endpoint].ts` — API handler
- `src/tests/[test].spec.ts` — test suite

## Validation Commands
```bash
npm run test -- --coverage
npm run lint
npm run build
```

## Jira Sub-tasks
| Sub-task Key | Title | Estimate |
|--------------|-------|----------|
| PROJ-XXX-1 | Data model implementation | 2h |
| PROJ-XXX-2 | API endpoint | 3h |
| PROJ-XXX-3 | Unit tests | 2h |

## Definition of Done
- [ ] All tests pass
- [ ] Coverage > 80%
- [ ] No lint errors
- [ ] PR reviewed and approved
- [ ] Jira ticket transitioned to Done
```

### Release Document (`release-vX.X.X.md`)

```markdown
# Release: v[X.X.X]
## Date: [YYYY-MM-DD]
## Jira Release Key: [PROJ-VERSION]

## What's New
### Features
- [PROJ-XXX] Feature description

### Bug Fixes
- [PROJ-YYY] Bug description

### Improvements
- [PROJ-ZZZ] Improvement description

## Breaking Changes
[List any breaking changes with migration guide]

## Deployment Checklist
- [ ] All CI checks pass
- [ ] Staging deployment verified
- [ ] Database migrations run
- [ ] Feature flags configured
- [ ] Rollback plan documented
- [ ] Stakeholders notified
- [ ] Monitoring alerts active

## Rollback Plan
[Steps to revert if issues found]

## Known Issues
[Any known limitations in this release]
```

---

## 8. Prompt Template Structure

Every command file follows this structure:

```markdown
# [Command Name] — SDLC Phase [N]: [Phase Name]

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/[relevant-artifacts]/` — existing project artifacts
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — [Description of what the argument represents]

## Task
[Detailed instruction of what Claude should do]

### Step 1: [Action]
[Specific instructions]

### Step 2: [Action]
[Specific instructions]

### Step N: Jira Sync
[Always the last step — push artifacts to Jira via API or provide
the curl/API call the user needs to execute]

## Output
Produce the following artifacts:
1. **Local file:** `.claude/SDLCs/[folder]/[filename].md`
2. **Jira action:** [Specific API action and payload]
3. **Summary:** A brief confirmation of what was created/updated

## Validation
Before finishing, confirm:
- [ ] Artifact file written to correct path
- [ ] Jira fields populated correctly
- [ ] All links between issues established
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `[/sdlc-next-command] [args]`
```

---

## 9. Jira API Mapping

### Create an Epic from `/sdlc-epic`

```json
POST /rest/api/3/issue
{
  "fields": {
    "project": { "key": "PROJ" },
    "summary": "[Epic title from idea]",
    "description": {
      "type": "doc", "version": 1,
      "content": [{ "type": "paragraph", "content": [
        { "type": "text", "text": "[Epic description]" }
      ]}]
    },
    "issuetype": { "name": "Epic" },
    "customfield_10014": "[Epic Name]",
    "priority": { "name": "High" },
    "labels": ["sdlc-plugin", "phase-1-planning"]
  }
}
```

### Bulk Create Stories from `/sdlc-stories`

```json
POST /rest/api/3/issue/bulk
{
  "issueUpdates": [
    {
      "fields": {
        "project": { "key": "PROJ" },
        "summary": "As a [user], I want [goal] so that [benefit]",
        "issuetype": { "name": "Story" },
        "customfield_10016": [story_points],
        "customfield_10014": "[Epic Key]",
        "description": { ... },
        "labels": ["sdlc-plugin"]
      }
    }
  ]
}
```

### Transition Issue from `/sdlc-implement`

```json
POST /rest/api/3/issue/{issueKey}/transitions
{
  "transition": { "id": "[transition_id_for_in_progress]" }
}
```

### Create Sprint from `/sdlc-sprint`

```json
POST /rest/agile/1.0/sprint
{
  "name": "Sprint [N] — [Epic Name]",
  "startDate": "[ISO date]",
  "endDate": "[ISO date + 2 weeks]",
  "originBoardId": [board_id],
  "goal": "[Sprint goal derived from selected stories]"
}
```

### JQL Queries Used

```jql
-- Get all stories in an Epic
project = PROJ AND "Epic Link" = PROJ-XXX AND issuetype = Story

-- Get Sprint backlog
project = PROJ AND sprint = "Sprint N" AND status != Done

-- Get all blockers
project = PROJ AND issueType = Story AND priority = Blocker AND status != Done

-- Get release candidates
project = PROJ AND fixVersion = "v1.2.0" AND status = "In Review"
```

---

## 10. Configuration & Setup

### Prerequisites

```bash
# 1. Jira API token (generate at: https://id.atlassian.com/manage-profile/security/api-tokens)
export JIRA_API_TOKEN="<your-atlassian-api-token>"
export JIRA_EMAIL="hmoro.dev@outlook.com"
export JIRA_BASE_URL="https://hmorodev.atlassian.net"
export JIRA_PROJECT_KEY="MA"
```

### Initial Setup Steps

1. **Clone the plugin commands** into your project:
   ```bash
   mkdir -p .claude/commands/sdlc
   mkdir -p .claude/SDLCs/{ideas,prds,epics,plans/completed,designs,test-reports,releases,retros}
   mkdir -p .claude/agents
   mkdir -p .claude/settings
   ```

2. **Configure Jira** — fill in `.claude/settings/jira-config.json` with your project details.

3. **Add to `CLAUDE.md`** at project root:
   ```markdown
   ## SDLC Plugin
   This project uses the SDLC Claude plugin. All development follows the phases
   defined in SPEC.md. Every code change must reference a Jira ticket.

   - Jira project: PROJ
   - Board: https://your-org.atlassian.net/jira/software/projects/PROJ/boards
   - Commands: /sdlc-* (see SPEC.md for full reference)
   ```

4. **Verify Jira connectivity** before first use:
   ```bash
   curl -u $JIRA_EMAIL:$JIRA_API_TOKEN \
     "$JIRA_BASE_URL/rest/api/3/myself" | jq '.displayName'
   ```

---

## 11. Directory Structure

```
.claude/
├── commands/
│   └── sdlc/
│       ├── phase-1-planning/
│       │   ├── sdlc-idea.md         # /sdlc-idea command
│       │   ├── sdlc-epic.md         # /sdlc-epic command
│       │   └── sdlc-roadmap.md      # /sdlc-roadmap command
│       ├── phase-2-requirements/
│       │   ├── sdlc-prd.md          # /sdlc-prd command
│       │   ├── sdlc-stories.md      # /sdlc-stories command
│       │   └── sdlc-acceptance.md   # /sdlc-acceptance command
│       ├── phase-3-design/
│       │   ├── sdlc-design.md       # /sdlc-design command
│       │   └── sdlc-arch.md         # /sdlc-arch command
│       ├── phase-4-development/
│       │   ├── sdlc-plan.md         # /sdlc-plan command
│       │   ├── sdlc-implement.md    # /sdlc-implement command
│       │   └── sdlc-commit.md       # /sdlc-commit command
│       ├── phase-5-testing/
│       │   ├── sdlc-test.md         # /sdlc-test command
│       │   ├── sdlc-bug.md          # /sdlc-bug command
│       │   └── sdlc-qa.md           # /sdlc-qa command
│       ├── phase-6-deployment/
│       │   ├── sdlc-release.md      # /sdlc-release command
│       │   └── sdlc-deploy.md       # /sdlc-deploy command
│       ├── phase-7-maintenance/
│       │   ├── sdlc-retro.md        # /sdlc-retro command
│       │   ├── sdlc-monitor.md      # /sdlc-monitor command
│       │   └── sdlc-hotfix.md       # /sdlc-hotfix command
│       └── utils/
│           ├── sdlc-status.md       # /sdlc-status command
│           ├── sdlc-sync-jira.md    # /sdlc-sync-jira command
│           └── sdlc-sprint.md       # /sdlc-sprint command
│
├── agents/
│   ├── jira-connector.md            # Handles all Jira API calls
│   ├── requirements-analyst.md      # Specializes in PRDs & stories
│   ├── tech-architect.md            # Handles design & ADRs
│   ├── qa-engineer.md               # Test plans & QA checklists
│   └── release-manager.md           # Release notes & deploy checklists
│
├── SDLCs/                           # All generated artifacts
│   ├── ideas/
│   ├── prds/
│   ├── epics/
│   ├── plans/
│   │   └── completed/
│   ├── designs/
│   ├── test-reports/
│   ├── releases/
│   └── retros/
│
└── settings/
    └── jira-config.json             # Jira connection config

SPEC.md                              # This file
CLAUDE.md                            # Project conventions for Claude
```

---

## Appendix — Command Quick Reference Card

```
PHASE 1 — PLANNING
  /sdlc-idea    "your idea"          → Refine idea → Jira Initiative
  /sdlc-epic    [idea-file|KEY]      → Create Epic → Jira Epic
  /sdlc-roadmap [epic-key] [N]       → Roadmap → Jira timeline

PHASE 2 — REQUIREMENTS
  /sdlc-prd     [epic-key|idea-file] → Full PRD doc
  /sdlc-stories [prd-file|epic-key]  → User Stories → Jira bulk create
  /sdlc-acceptance [story-key]       → Gherkin criteria → Jira field

PHASE 3 — DESIGN
  /sdlc-design  [story-key|feature]  → Technical design doc
  /sdlc-arch    [design-file|key]    → ADR → Jira comment

PHASE 4 — DEVELOPMENT
  /sdlc-plan    [story-key|design]   → Impl plan → Jira sub-tasks
  /sdlc-implement [plan-file|key]    → Execute code → Jira In Progress
  /sdlc-commit  [story-key]          → Commit msg + PR → Jira link

PHASE 5 — TESTING
  /sdlc-test    [story-key|file]     → Test suite → Jira QA task
  /sdlc-bug     "description"        → Bug ticket → Jira Bug
  /sdlc-qa      [story-key|release]  → QA checklist → Jira In Review

PHASE 6 — DEPLOYMENT
  /sdlc-release [version] [epic-key] → Release notes → Jira Release
  /sdlc-deploy  [release] [env]      → Deploy check → Jira Done

PHASE 7 — MAINTENANCE
  /sdlc-retro   [sprint-id|release]  → Retrospective → Jira Tasks
  /sdlc-monitor [release-key]        → Monitor check → Jira Bugs
  /sdlc-hotfix  [bug-key]            → Hotfix flow → Expedited deploy

UTILITIES
  /sdlc-status  [epic|sprint]        → SDLC dashboard + next action
  /sdlc-sync-jira [artifact-file]    → Push any artifact → Jira
  /sdlc-sprint  [epic-key] [N]       → Sprint plan → Jira Sprint
```

---

*SPEC.md — SDLC Claude Plugin · Version 1.0.0 · Generated 2026-03-16*
