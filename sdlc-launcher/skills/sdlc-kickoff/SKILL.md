---
name: sdlc-kickoff
description: "SDLC Pipeline: From raw idea to full Jira backlog with project brief, epic, PRD, user stories, and Sprint 1 — in one command"
---

# sdlc-kickoff — SDLC Full Pipeline: Idea → Ready Backlog

## MCP Tools
- **Jira** (`jira` MCP): `jira_create_issue`, `jira_add_comment`, `jira_create_sprint`, `jira_add_issues_to_sprint`, `jira_search_issues`
- **GitHub** (`github` MCP): `push_files`, `create_branch` — upload project brief to repo
- **Confluence** (via `jira` MCP / mcp-atlassian): `confluence_create_page` — publish project brief as Confluence page

## Overstory — Parallel Artifact Generation
This skill uses **Overstory multi-agent parallelism** for independent generation phases.

Once the Epic is created (Step 4), spawn parallel Overstory agents:

```bash
# Agent A — PRD generation
overstory spawn \
  --branch "overstory/kickoff-prd-{epicKey}" \
  --task "Generate PRD for epic {epicKey} using .claude/SDLCs/briefs/brief-{slug}.md and .claude/SDLCs/epics/epic-{epicKey}.md. Output to .claude/SDLCs/prds/prd-{epicKey}.md. Follow SPEC.MD PRD format." \
  --worktree

# Agent B — User Stories + Acceptance Criteria
overstory spawn \
  --branch "overstory/kickoff-stories-{epicKey}" \
  --task "Generate user stories with acceptance criteria for epic {epicKey}. Read .claude/SDLCs/briefs/brief-{slug}.md. Output one file per story to .claude/SDLCs/stories/. Follow SPEC.MD story format. Include story points and priority." \
  --worktree

# Agent C — Technical Architecture ADRs (if technical idea)
overstory spawn \
  --branch "overstory/kickoff-arch-{epicKey}" \
  --task "Generate Architecture Decision Records for epic {epicKey}. Read brief and identify top 3 architectural decisions. Output to .claude/SDLCs/designs/adr-{epicKey}-00{N}.md." \
  --worktree
```

Wait for all agents to complete before proceeding to Step 6.
If any agent fails → merge available results, flag missing artifacts, continue.

## Mulch & Seeds

### Session Start
```bash
mulch prime --context   # load existing project conventions
sd prime                # load issue tracking context
```

### During kickoff
```bash
mulch search "project type"         # check if similar project was done before
sd create --title "Kickoff: {idea}" --type task --priority 1
sd update <id> --status in_progress
```

### Session Close
```bash
mulch learn
mulch record sdlc-commands --type convention \
  --description "Kickoff pattern for {project type}: {key decisions made}"
mulch sync
sd close <kickoff-id>
sd sync
```

## Context Loading
1. Run session start protocol (Mulch + Seeds above)
2. Read `.claude/settings/jira-config.json` — project key, board ID, Jira URL
3. Read `.claude/SDLCs/` — check existing artifacts to avoid numbering conflicts
4. Read `CLAUDE.md` — project conventions and tech stack constraints

## Input
$ARGUMENTS — Raw idea description (free text). Examples:
- `"A task tracker for personal finance"`
- `"An AI-powered code review tool for GitHub PRs"`
- `"Mobile app for tracking gym workouts"`

## Task

---

### Step 0: Jira Pull Check

**Ask the user:**
```
Before we start — have you made any manual changes in Jira since your last session?
(yes / no)
```

If **yes** → run `sdlc-pull` flow first (fetch changes, update local artifacts, report diff), then continue.
If **no** → proceed to Step 1.

---

### Step 1: Clarify & Scope the Idea

Extract from $ARGUMENTS (ask follow-up questions if critical info is missing):

| Field | Question to ask if not in $ARGUMENTS |
|-------|--------------------------------------|
| Problem statement | "What problem does this solve?" |
| Target users | "Who is the primary user?" |
| Core value | "What's the #1 thing it must do well?" |
| Platform | "Web, mobile, API, or all?" |
| Scale | "MVP or production-grade from day one?" |
| Team size | "Solo, small team (2-5), or larger?" |

Do **not** ask more than 3 follow-up questions. Make reasonable assumptions for the rest and state them clearly.

---

### Step 2: Generate Project Brief

Create `.claude/SDLCs/briefs/brief-{slug}.md`:

```markdown
# Project Brief: {Project Name}

## Meta
- Created: {date}
- Status: Draft
- Epic: {epicKey} (created in Step 3)

## Vision
{1-2 sentence product vision}

## Problem Statement
{What problem exists, who suffers from it, what it costs them}

## Target Users
{Primary persona: role, goals, frustrations}
{Secondary persona if applicable}

## Goals & Success Metrics
| Goal | Metric | Target |
|------|--------|--------|
| {goal} | {how measured} | {threshold} |

## Feature Scope
### In Scope (MVP)
- {feature 1}
- {feature 2}

### Out of Scope
- {deferred feature}

## High-Level Architecture
{2-3 sentences on tech approach — platform, data model, integrations}

## Timeline
| Phase | Duration | Milestone |
|-------|----------|-----------|
| Foundation | {N weeks} | {deliverable} |
| Core features | {N weeks} | {deliverable} |
| Polish & launch | {N weeks} | {deliverable} |

## Risks & Assumptions
- Risk: {description} | Mitigation: {action}
- Assumption: {what we're assuming to be true}

## Stakeholders
- Product Owner: {name or TBD}
- Tech Lead: {name or TBD}
```

**Upload the brief:**

```
# GitHub MCP: push_files
# Push to: docs/project-brief-{slug}.md in the project repository
# Commit message: "docs: add project brief for {Project Name}"
```

```
# Confluence MCP (mcp-atlassian): confluence_create_page
# Space: project space or default
# Title: "Project Brief — {Project Name}"
# Parent: Epic page (link after epic creation in Step 3)
# Body: full brief content
```

---

### Step 3: Create Jira Epic

```
# MCP: jira_create_issue
# issuetype: Epic
# summary: {Project Name}
# description: {vision + problem statement from brief}
# priority: High
# labels: ["sdlc-plugin", "kickoff"]
```

Save the returned Epic key (e.g., `MA-2`). Update the brief's `Epic:` field.

---

### Step 4: Spawn Parallel Overstory Agents

Spawn 3 agents simultaneously (see Overstory section above):
- **Agent A**: PRD → `.claude/SDLCs/prds/prd-{epicKey}.md`
- **Agent B**: User stories + AC → `.claude/SDLCs/stories/story-{epicKey}-{N}.md`
- **Agent C**: Architecture ADRs → `.claude/SDLCs/designs/adr-{epicKey}-00{N}.md`

While agents run, proceed to Step 5.

---

### Step 5: Wait & Merge Agent Results

Once all Overstory agents complete:
1. Read all generated files from each agent's branch
2. Merge to main (or copy outputs if worktree-based)
3. Validate: PRD exists, ≥3 stories exist, ≥1 ADR exists
4. If any missing → generate that artifact in main agent as fallback

---

### Step 6: Build Product Backlog

Read all generated stories. For each story:

```
# MCP: jira_create_issue
# issuetype: Story
# summary: {story title}
# description: {user story + acceptance criteria}
# epicLink: {epicKey}
# priority: {derived from story priority field}
# story_points: {from story file}
# labels: ["sdlc-plugin", "backlog"]
```

Print backlog table as stories are created:
```
PRODUCT BACKLOG — {Project Name} ({epicKey})
─────────────────────────────────────────────
  MA-3  Create a financial task              [Critical] [3 pts] ✓
  MA-4  Edit and delete tasks                [High]     [2 pts] ✓
  MA-5  Complete task with timestamp         [Critical] [2 pts] ✓
  ...
─────────────────────────────────────────────
Total: {N} stories | {X} points
```

---

### Step 7: Plan Sprint 1

From the backlog, select Sprint 1 stories:
- Prioritize: Critical > High > Medium
- Respect dependency order (foundation stories first)
- Fit within capacity: 1 dev × 10 days × 0.7 = ~14 story points default
  (ask user to confirm team size if not in context)

```
# MCP: jira_create_sprint
# name: "Sprint 1 — Foundation"
# startDate: {next Monday}
# endDate: {+14 days}
# originBoardId: {from jira-config}
# goal: {one sentence sprint goal}
```

```
# MCP: jira_add_issues_to_sprint
# Add all committed Sprint 1 story keys
```

---

### Step 8: Notify Team

Add a comment on the Jira Epic summarising the kickoff result:

```
# MCP: jira_add_comment
# Issue: {epicKey}
# Body: "Kickoff complete. {N} stories in backlog. Sprint 1 created.
# Project brief: {GitHub URL} | Confluence: {Confluence URL}"
```

---

### Step 9: Session Close Protocol

```bash
mulch learn
mulch record sdlc-commands --type pattern \
  --description "Kickoff for {project type}: brief → epic → parallel PRD+stories+arch → backlog → sprint"
mulch sync

sd close <kickoff-seeds-issue-id>
sd sync
```

---

## Output
| Artifact | Location | Jira |
|----------|----------|------|
| Project Brief | `.claude/SDLCs/briefs/brief-{slug}.md` | Confluence page + GitHub `/docs/` |
| Epic | `.claude/SDLCs/epics/epic-{epicKey}.md` | Epic ticket |
| PRD | `.claude/SDLCs/prds/prd-{epicKey}.md` | Comment on epic |
| User Stories (N) | `.claude/SDLCs/stories/story-{epicKey}-{N}.md` | Story ticket per story |
| ADRs | `.claude/SDLCs/designs/adr-{epicKey}-00{N}.md` | — |
| Sprint 1 | — | Sprint created, stories assigned |

## Validation
- [ ] Project brief written locally + uploaded to GitHub + Confluence
- [ ] Epic created in Jira with correct fields
- [ ] PRD generated and linked to epic
- [ ] ≥3 user stories with acceptance criteria, each with a Jira ticket
- [ ] ≥1 ADR generated
- [ ] Sprint 1 created in Jira with committed stories
- [ ] Kickoff summary comment added to Jira Epic
- [ ] Mulch and Seeds session close protocol run

## Next Step
`/sdlc-plan {first-story-key}` — create implementation plan for the first Sprint 1 story
