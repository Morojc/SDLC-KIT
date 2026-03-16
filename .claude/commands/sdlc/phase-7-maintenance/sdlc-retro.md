# sdlc-retro — SDLC Phase 7: Maintenance

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/releases/` — release documents (if a release-based retro)
- `.claude/SDLCs/plans/completed/` — completed implementation plans for context
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — `[sprint-id or release-key]`

- `sprint-id` — Jira sprint identifier (e.g., `PROJ-SPRINT-1`) or sprint name
- `release-key` — version/release identifier (e.g., `v1.2.0`) for release retrospectives

## Task
Generate a structured sprint or release retrospective and create Jira Tasks for all action items identified.

### Step 1: Gather Sprint/Release Data
Query Jira for sprint or release metrics:

**For sprint retrospectives:**
```
GET {JIRA_BASE_URL}/rest/agile/1.0/sprint/{sprintId}
GET {JIRA_BASE_URL}/rest/agile/1.0/sprint/{sprintId}/issue
```

**For release retrospectives:**
```
GET {JIRA_BASE_URL}/rest/api/3/search?jql=project={PROJECT_KEY} AND fixVersion="{release-key}"
```

Collect:
- Total stories planned vs. completed
- Bug count introduced and resolved
- Story points completed vs. committed
- Average cycle time per story
- Any blockers or impediments encountered

### Step 2: Facilitate Retrospective Structure
Structure the retrospective across four categories:

**What Went Well**
- Team collaboration and communication highlights
- Technical wins (architecture decisions, tooling improvements)
- Process successes (CI speed, review quality)
- Delivery achievements

**What to Improve**
- Bottlenecks or slowdowns identified
- Technical debt accumulated
- Process gaps or friction points
- Testing or quality gaps

**Action Items**
- Specific, measurable improvements to implement
- Assign owner and due date for each
- Prioritize by impact

**Team Health**
- Morale and engagement observations
- Workload balance assessment
- Knowledge sharing gaps

### Step 3: Write Retrospective Document
Save to `.claude/SDLCs/retros/retro-{sprint-or-release}.md`:

```markdown
# Retrospective: {Sprint/Release Name}
## Date: {YYYY-MM-DD}
## Participants: {list from Jira assignees}

## Metrics
| Metric | Target | Actual |
|--------|--------|--------|
| Stories completed | {planned} | {actual} |
| Story points | {committed} | {delivered} |
| Bugs introduced | — | {count} |
| Bugs resolved | — | {count} |

## What Went Well
- {item 1}
- {item 2}

## What to Improve
- {item 1}
- {item 2}

## Action Items
| # | Action | Owner | Due Date | Jira Task |
|---|--------|-------|----------|-----------|
| 1 | {action} | {owner} | {date} | {PROJ-XXX} |

## Team Health
{Overall team health assessment}

## Next Sprint Focus
{Key themes for the next sprint based on retrospective findings}
```

### Step 4: Jira Sync — Create Action Item Tasks
For each action item identified, create a Jira Task in the next sprint:

```
POST {JIRA_BASE_URL}/rest/api/3/issue
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "fields": {
    "project": { "key": "{PROJECT_KEY}" },
    "summary": "Retro Action: {action description}",
    "description": {
      "type": "doc", "version": 1,
      "content": [{ "type": "paragraph", "content": [
        { "type": "text", "text": "Retrospective action item from {sprint/release}. Context: {action context}" }
      ]}]
    },
    "issuetype": { "name": "Task" },
    "priority": { "name": "Medium" },
    "labels": ["sdlc-plugin", "retro-action-item"]
  }
}
```

Update the retro document with the created Jira task keys.

## Output
1. **Local file:** `.claude/SDLCs/retros/retro-{sprint-or-release}.md`
2. **Jira action:** Create Task issues for each action item via `POST /rest/api/3/issue`
3. **Summary:** Retrospective complete — N action items created as Jira Tasks

## Validation
- [ ] Retrospective document written to `.claude/SDLCs/retros/`
- [ ] All action items created as Jira Tasks
- [ ] Retro document updated with Jira task keys
- [ ] Sprint/release metrics included
- [ ] Next command printed

## Next Step
After completion, suggest: `/sdlc-sprint {epic-key} {next-sprint-number}`
