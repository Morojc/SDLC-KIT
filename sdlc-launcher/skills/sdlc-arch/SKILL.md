---
name: sdlc-arch
description: "SDLC Phase 3: Create Architecture Decision Records with trade-off analysis"
---

# sdlc-arch — SDLC Phase 3: Design

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
- `.claude/SDLCs/designs/` — technical design documents
- `.claude/SDLCs/epics/` — epic for business and architectural context
- `.claude/SDLCs/prds/` — PRD for non-functional requirements
- `CLAUDE.md` (if exists) — project conventions, existing architecture decisions

## Input
$ARGUMENTS — Either:
- A path to a design file (e.g., `design-PROJ-010.md` or `.claude/SDLCs/designs/design-PROJ-010.md`)
- A Jira Epic or Story key (e.g., `PROJ-002`)

## Task
Produce an Architecture Decision Record (ADR) with trade-off analysis. An ADR documents the architectural decision being made, the context forcing it, the alternatives considered, and the consequences of the chosen approach. Add the ADR to Jira.

### Step 1: Load Design and Context
If $ARGUMENTS is a file path, read the design document from `.claude/SDLCs/designs/`.
If $ARGUMENTS is a Jira key, fetch the issue and find the associated design document:

```
# MCP: jira_create_issue
# project.key, summary, issuetype, description, priority, labels
```

Also read `CLAUDE.md` for any existing architectural constraints or prior ADRs.

### Step 2: Identify Architectural Decisions
From the design document, extract:
- The core architectural question(s) being answered
- System quality attributes at stake (performance, scalability, maintainability, security)
- Constraints from the tech stack, team, or business
- Alternatives that were or could be considered

Each significant architectural choice gets its own ADR section.

### Step 3: Write the ADR Document
Create `.claude/SDLCs/designs/adr-[EPIC-or-STORY-KEY]-[N].md`:

```markdown
# Architecture Decision Record: [Decision Title]

## ADR Reference
- ADR Number: [sequential number, e.g., ADR-001]
- Story/Epic Key: [PROJ-XXX]
- Date: [today's date]
- Status: Proposed | Accepted | Deprecated | Superseded
- Deciders: [tech lead, team, or stakeholders who must approve]
- Supersedes: [prior ADR key if applicable]

## Context

### Problem Statement
[What architectural problem are we solving? Why does this decision need to be made now?]

### Constraints
- [Technical constraint — e.g., must use existing Postgres instance]
- [Business constraint — e.g., budget, timeline, team expertise]
- [Non-functional constraint — e.g., < 100ms latency requirement]

### Quality Attributes at Stake
| Attribute | Priority | Rationale |
|-----------|----------|-----------|
| Performance | High | [why] |
| Scalability | Medium | [why] |
| Maintainability | High | [why] |
| Security | High | [why] |

## Decision

### Chosen Approach
**[Name of chosen approach]**

[Detailed description of what is being decided. Be specific enough that a new engineer understands exactly what architectural pattern/technology/structure was chosen and how it works.]

### Rationale
[Why was this the best option given the constraints and quality attributes? Reference specific trade-offs.]

## Alternatives Considered

### Alternative 1: [Name]
**Description:** [What this approach would look like]
**Pros:**
- [advantage 1]
- [advantage 2]
**Cons:**
- [disadvantage 1]
- [disadvantage 2]
**Why rejected:** [Specific reason — not just "worse", but why for this context]

### Alternative 2: [Name]
**Description:** [What this approach would look like]
**Pros:**
- [advantage 1]
**Cons:**
- [disadvantage 1]
**Why rejected:** [Specific reason]

## Consequences

### Positive
- [Expected benefit 1]
- [Expected benefit 2]

### Negative / Trade-offs
- [Accepted trade-off 1 — and why it's acceptable]
- [Accepted trade-off 2]

### Risks
- Risk: [description] | Mitigation: [action]
- Risk: [description] | Mitigation: [action]

## Implementation Notes

### What Changes
- [Component or file that changes]
- [New component or service being added]

### Migration Path
[If this ADR replaces existing architecture, how do we get there incrementally?]

### Validation
How will we know this decision was correct?
- Metric: [what we'll measure]
- Target: [success threshold]
- Review Date: [when to revisit this decision]

## References
- [Link to design doc]
- [Link to relevant RFC, blog post, or prior art]
- [Link to Jira ticket]
```

### Step 4: Jira Sync — Add ADR as Comment on Epic/Story
Post the ADR summary to the associated Jira issue:

```
# MCP: jira_add_comment
# Post structured summary comment to the issue
```

If this ADR creates a Task for architectural validation, create it:

```
# MCP: jira_create_issue
# fields: project.key, summary, issuetype, epicLink, priority, description, labels
```

## Output
1. **Local file:** `.claude/SDLCs/designs/adr-[KEY]-[N].md`
2. **Jira action:** Add ADR comment to Epic/Story; optionally create ADR Review Task
3. **Summary:** Print ADR number, decision title, alternatives count, and status

## Validation
Before finishing, confirm:
- [ ] ADR file written to `.claude/SDLCs/designs/`
- [ ] Context, constraints, and quality attributes documented
- [ ] At least 2 alternatives considered with specific rejection rationale
- [ ] Consequences section covers both positive and negative outcomes
- [ ] Validation metrics defined
- [ ] Jira comment added
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-plan [STORY-KEY]`
