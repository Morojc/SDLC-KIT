---
name: sdlc-sprint
description: "SDLC Utility: Plan a sprint by selecting stories and calculating team capacity"
---

# sdlc-sprint — SDLC Utility: Sprint Planning

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
- `.claude/settings/jira-config.json` — Jira project settings (board ID, sprint settings)
- `.claude/SDLCs/epics/` — epic artifacts for context
- `.claude/SDLCs/plans/` — implementation plans for effort estimation
- `CLAUDE.md` (if exists) — project conventions and team capacity

## Input
$ARGUMENTS — `[epic-key] [sprint-number]`

- `epic-key` — Jira Epic key to pull stories from (e.g., `PROJ-002`)
- `sprint-number` — Sprint number to plan (e.g., `1`, `2`)

## Task
Plan the sprint by selecting and assigning stories, estimating team capacity, identifying risks, and creating the sprint in Jira with all assigned stories moved into it.

### Step 1: Load Epic Backlog
Fetch all unassigned/backlog stories for the epic:
```
GET {JIRA_BASE_URL}/rest/api/3/search?jql=project={PROJECT_KEY} AND "Epic Link"={epic-key} AND sprint is EMPTY AND status="To Do" ORDER BY priority DESC
```

Retrieve story details: summary, story points, priority, assignee, acceptance criteria.

### Step 2: Assess Team Capacity
Based on available information (from `CLAUDE.md`, jira-config, or prompted input):
- Team size: {N} developers
- Sprint duration: 2 weeks (default)
- Working days per developer: 10 days
- Capacity factor: 0.7 (accounts for meetings, reviews, overhead)
- **Total capacity:** {N} developers × 10 days × 0.7 = {X} developer-days

Convert to story points using team's average velocity (default assumption: 1 story point ≈ 0.5 developer-days, or use historical velocity if available).

**Estimated sprint capacity:** {Y} story points

### Step 3: Prioritize and Select Stories
Rank backlog stories and select those fitting within sprint capacity:

Priority order:
1. Blocker / Critical priority items
2. Dependencies required by other teams
3. High business value (from epic/PRD)
4. Smaller stories first (reduce WIP risk)

Generate a sprint candidate table:
| Story Key | Summary | Points | Priority | Risk | Include? |
|-----------|---------|--------|----------|------|----------|
| PROJ-XXX | {summary} | {pts} | High | Low | ✅ |
| PROJ-YYY | {summary} | {pts} | Medium | Medium | ✅ |
| PROJ-ZZZ | {summary} | {pts} | Low | High | ⚠️ Stretch |

Selected: {total} stories / {total_points} points (vs. capacity: {capacity} points)

### Step 4: Risk Assessment
For each selected story, assess risks:
- **Dependencies:** Does this story depend on another not yet started?
- **Complexity:** Any unknowns or new technology?
- **External blockers:** Waiting on third parties, design approvals, etc.
- **Team knowledge:** Does the assigned developer know this area?

Summarize top 3 sprint risks with mitigation strategies.

### Step 5: Generate Sprint Plan Document
Output the sprint plan in the chat:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPRINT {N} PLAN — {Epic Name}
Epic: {epic-key} | Dates: {start} → {end}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SPRINT GOAL
{One sentence describing what this sprint delivers}

CAPACITY
  Team size:     {N} devs
  Working days:  10
  Capacity:      {X} story points

COMMITTED STORIES ({count} stories / {points} pts)
  {PROJ-XXX} — {summary} [{assignee}] ({pts} pts)
  ...

STRETCH GOALS (if capacity allows)
  {PROJ-ZZZ} — {summary} [{assignee}] ({pts} pts)

TOP RISKS
  1. {Risk description} → Mitigation: {action}
  2. {Risk description} → Mitigation: {action}

DEFINITION OF DONE
  - All acceptance criteria met
  - Tests passing (unit + integration)
  - PR reviewed and merged
  - Jira ticket in Done status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 6: Jira Sync — Create Sprint
```
POST {JIRA_BASE_URL}/rest/agile/1.0/sprint
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "name": "Sprint {N} — {Epic Name}",
  "startDate": "{YYYY-MM-DDTHH:MM:SS.000Z}",
  "endDate": "{YYYY-MM-DDTHH:MM:SS.000Z}",
  "originBoardId": {board_id},
  "goal": "{Sprint goal text}"
}
```

### Step 7: Move Stories into Sprint

```
# MCP: jira_add_issues_to_sprint
# Add all committed story keys to the sprint
```

### Step 8: Create Sprint Branch on GitHub

Once the Jira sprint is created, create the corresponding Git branch:

```
# MCP: github — create_branch
# branch: sprint/{N}-{goal-slug}
#   N    = sprint number (e.g. 1)
#   slug = sprint goal lowercased, spaces→hyphens, max 30 chars
#          e.g. "Stand up auth and core CRUD" → "stand-up-auth-and-core-crud"
# base:  main
```

Then for each **dev story** committed to this sprint, create its story branch:

```
# MCP: github — create_branch (repeat per dev story)
# branch: story/{KEY}-{title-slug}
#   KEY  = Jira story key (e.g. MA-3)
#   slug = story title lowercased, hyphens, max 30 chars
# base:  sprint/{N}-{goal-slug}   ← branch from sprint, NOT from main
```

**Dev story detection** (use git-manager agent rules):
- Skip non-dev stories: design, research, spike, QA-only
- Create branch for: feature, implementation, bug, chore with code output

Print branch summary:
```
GIT BRANCHES CREATED
──────────────────────────────────────────
  sprint/1-foundation              (from main)
  story/MA-3-create-task           (from sprint/1-foundation)
  story/MA-5-complete-task         (from sprint/1-foundation)
  story/MA-10-data-persistence     (from sprint/1-foundation)
  story/MA-12-authentication       (from sprint/1-foundation)
  [MA-9 skipped — UX story, no branch]
──────────────────────────────────────────
```

## Output
1. **Local file:** Sprint plan saved to `.claude/SDLCs/plans/sprint-{N}-plan.md`
2. **Jira actions:** Sprint created; stories assigned to sprint
3. **GitHub actions:** `sprint/{N}-{slug}` branch + one `story/{KEY}-{slug}` branch per dev story
4. **Summary:** Sprint {N} created — {N} stories / {points} pts — {B} branches created

## Validation
- [ ] Team capacity calculated
- [ ] Stories selected within capacity limits
- [ ] Risks identified with mitigations
- [ ] Sprint created in Jira with correct dates and goal
- [ ] All committed stories moved to sprint in Jira
- [ ] `sprint/{N}-{slug}` branch created from `main`
- [ ] One `story/{KEY}-{slug}` branch created per dev story, based from sprint branch
- [ ] Non-dev stories correctly skipped (no branch created)
- [ ] Sprint plan output clearly presented

## Next Step
After sprint is created, begin implementation:
`/sdlc-plan {first-story-key}`
