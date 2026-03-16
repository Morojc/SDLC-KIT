# sdlc-stories — SDLC Phase 2: Requirements

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/prds/` — PRD documents
- `.claude/SDLCs/epics/` — epic documents for business context
- `CLAUDE.md` (if exists) — project conventions and tech stack

## Input
$ARGUMENTS — Either:
- A path to a PRD file (e.g., `prd-PROJ-002.md` or `.claude/SDLCs/prds/prd-PROJ-002.md`)
- A Jira Epic key (e.g., `PROJ-002`)

## Task
Decompose a PRD into Jira User Stories with story points estimate. Each story maps to one functional requirement or a coherent user-facing feature slice. Bulk create all stories in Jira under the Epic.

### Step 1: Load PRD Content
If $ARGUMENTS is a file path, read the PRD from `.claude/SDLCs/prds/`.
If $ARGUMENTS is a Jira key, read the Epic and its description, then fetch the local PRD file if it exists:

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/[EPIC-KEY]" | jq '.fields | {summary, description}'
```

### Step 2: Decompose into Stories
Apply INVEST criteria (Independent, Negotiable, Valuable, Estimable, Small, Testable):
- Each story should be completable in 1–3 days
- Format: "As a [role], I want [capability] so that [benefit]"
- Estimate story points using Fibonacci sequence: 1, 2, 3, 5, 8, 13
- Assign priority: Critical / High / Medium / Low
- Identify dependencies between stories

Produce 5–15 stories from the PRD functional requirements.

### Step 3: Write Story Files
Create one file per story in `.claude/SDLCs/stories/story-[EPIC-KEY]-[N].md`:

```markdown
# Story: [Story Title]

## Details
- Story Key: [PROJ-XXX — fill after creation]
- Epic Key: [EPIC-KEY]
- Priority: [Critical/High/Medium/Low]
- Story Points: [1/2/3/5/8/13]
- Status: To Do

## User Story
As a [role],
I want [capability],
So that [benefit].

## Acceptance Criteria
- Given [context], When [action], Then [outcome]
- Given [context], When [action], Then [outcome]

## Technical Notes
[Any implementation hints or constraints relevant to this story]

## Dependencies
- Depends on: [story key or system]
- Blocks: [story key or system]
```

### Step 4: Update PRD Story Summary Table
Update `.claude/SDLCs/prds/prd-[EPIC-KEY].md` User Stories Summary table with all generated story titles and point estimates.

### Step 5: Jira Sync — Bulk Create Stories
Construct the bulk create payload with all stories:

```bash
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/bulk" \
  -d '{
    "issueUpdates": [
      {
        "fields": {
          "project": { "key": "$JIRA_PROJECT_KEY" },
          "summary": "As a [role], I want [goal] so that [benefit]",
          "issuetype": { "name": "Story" },
          "customfield_10016": [story_points],
          "customfield_10014": "[EPIC-KEY]",
          "priority": { "name": "[priority]" },
          "description": {
            "type": "doc", "version": 1,
            "content": [{
              "type": "paragraph",
              "content": [{ "type": "text", "text": "Acceptance Criteria:\n[criteria list]" }]
            }]
          },
          "labels": ["sdlc-plugin", "phase-2-requirements"]
        }
      }
    ]
  }'
```

After bulk creation, update each story file with the assigned Jira key from the response.

Link dependent stories:

```bash
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issueLink" \
  -d '{
    "type": { "name": "Blocks" },
    "inwardIssue": { "key": "[BLOCKER-KEY]" },
    "outwardIssue": { "key": "[BLOCKED-KEY]" }
  }'
```

## Output
1. **Local files:** `.claude/SDLCs/stories/story-[EPIC-KEY]-[N].md` (one per story)
2. **Jira action:** Bulk create Stories under Epic; link dependencies
3. **Summary:** Print story count, total story points, and table of created stories with Jira keys

## Validation
Before finishing, confirm:
- [ ] All story files written to `.claude/SDLCs/stories/`
- [ ] Each story has INVEST-compliant user story and acceptance criteria
- [ ] Story points assigned to all stories
- [ ] Jira bulk create payload provided / executed
- [ ] PRD story summary table updated
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-acceptance [FIRST-STORY-KEY]`
