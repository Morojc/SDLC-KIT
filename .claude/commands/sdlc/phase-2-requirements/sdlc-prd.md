# sdlc-prd — SDLC Phase 2: Requirements

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

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/[EPIC-KEY]" | jq '.fields | {summary, description, status, labels}'
```

Also fetch any linked issues to understand existing context:

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/search?jql=project=$JIRA_PROJECT_KEY AND \"Epic Link\"=[EPIC-KEY]" \
  | jq '.issues[] | {key, fields: {summary, issuetype: .fields.issuetype.name}}'
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

```bash
# Update Epic description with PRD summary
curl -X PUT \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/[EPIC-KEY]" \
  -d '{
    "fields": {
      "description": {
        "type": "doc", "version": 1,
        "content": [{
          "type": "paragraph",
          "content": [{ "type": "text", "text": "[PRD executive summary]. Full PRD: .claude/SDLCs/prds/prd-[EPIC-KEY].md" }]
        }]
      }
    }
  }'

# Add PRD as comment
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/[EPIC-KEY]/comment" \
  -d '{
    "body": {
      "type": "doc", "version": 1,
      "content": [{
        "type": "paragraph",
        "content": [{ "type": "text", "text": "PRD generated and saved to .claude/SDLCs/prds/prd-[EPIC-KEY].md\n\nFunctional Requirements: [count]\nNon-Functional Requirements: [count]\nOpen Questions: [count]" }]
      }]
    }
  }'
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
