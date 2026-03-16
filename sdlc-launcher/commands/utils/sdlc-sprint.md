# sdlc-sprint — SDLC Utility: Sprint Planning

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
For each committed story, move it into the new sprint:
```
POST {JIRA_BASE_URL}/rest/agile/1.0/sprint/{sprintId}/issue
{
  "issues": ["{PROJ-XXX}", "{PROJ-YYY}", ...]
}
```

## Output
1. **Local file:** None (sprint plan output in chat; reference can be saved to `.claude/SDLCs/plans/sprint-{N}-plan.md` if desired)
2. **Jira action:** `POST /rest/agile/1.0/sprint` to create sprint; `POST /rest/agile/1.0/sprint/{id}/issue` to assign stories
3. **Summary:** Sprint {N} created — {N} stories / {points} points committed, {count} risks identified

## Validation
- [ ] Team capacity calculated
- [ ] Stories selected within capacity limits
- [ ] Risks identified with mitigations
- [ ] Sprint created in Jira with correct dates and goal
- [ ] All committed stories moved to sprint in Jira
- [ ] Sprint plan output clearly presented
- [ ] Next command printed

## Next Step
After sprint is created, begin implementation:
`/sdlc-plan {first-story-key}`
