# sdlc-plan — SDLC Phase 4: Development

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/designs/` — existing design documents (look for design matching the story key)
- `.claude/SDLCs/epics/` — epic context
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — A Jira story key (e.g. `PROJ-010`) or a path to a design file (e.g. `design-PROJ-010.md`)

## Task
Create a detailed, step-by-step implementation plan for the given story, breaking it into concrete sub-tasks with file targets, dependencies, and validation commands. Then create the sub-tasks in Jira and transition the parent story to In Progress.

### Step 1: Fetch Story Details
If the argument is a Jira key, retrieve the full issue via the Jira API:
```
GET $JIRA_BASE_URL/rest/api/3/issue/{issueKey}
Authorization: Basic base64($JIRA_EMAIL:$JIRA_API_TOKEN)
```
Extract: summary, description, acceptance criteria (`customfield_10016`), epic link (`customfield_10014`), assignee, sprint, story points.

If the argument is a design file, read it from `.claude/SDLCs/designs/` and extract the story key from it.

### Step 2: Analyze the Codebase
Explore the project structure to understand:
- Existing file patterns, naming conventions, directory layout
- Current test setup (test runner, coverage thresholds)
- Linting and build commands from `package.json` or equivalent
- Any existing models, APIs, or utilities the story will extend

### Step 3: Decompose into Implementation Phases
Break the story into 3–7 sequential or parallel implementation phases. For each phase:
- Assign a clear title
- List specific files to create or modify (use real paths based on codebase exploration)
- Note dependencies on other phases
- Flag whether it can run in parallel with other phases
- Estimate effort in hours (default: 1–4h per phase)

### Step 4: Define Validation Commands
For each phase, specify the exact commands to run to verify it:
- Unit tests: e.g. `bun test src/models/`
- Lint: e.g. `bun run lint`
- Type check: e.g. `bun run typecheck`
- Integration/e2e: e.g. `bun test --integration`
- Build check: e.g. `bun run build`

### Step 5: Write the Plan File
Write the implementation plan to `.claude/SDLCs/plans/plan-{storyKey}.md` using this schema:

```markdown
# Implementation Plan: {storyKey} — {Story Title}

## Jira Context
- Story Key: {storyKey}
- Epic: {epicKey}
- Sprint: {sprintName}
- Assignee: {assignee}
- Story Points: {points}

## Implementation Phases
| Phase | Task | Status | Depends On | Parallel? | Estimate |
|-------|------|--------|------------|-----------|----------|
| 1 | {task description} | pending | — | No | {Xh} |
| 2 | {task description} | pending | Phase 1 | No | {Xh} |
| 3 | {task description} | pending | Phase 2 | Yes | {Xh} |

## File Targets
- `{path/to/file.ts}` — {what it does}
- `{path/to/test.spec.ts}` — {what it tests}

## Validation Commands
```bash
{validation command for each phase}
```

## Jira Sub-tasks
| Sub-task Key | Title | Estimate |
|--------------|-------|----------|
| (created by Jira) | {Phase 1 title} | {Xh} |
| (created by Jira) | {Phase 2 title} | {Xh} |

## Definition of Done
- [ ] All tests pass with coverage > 80%
- [ ] No lint errors
- [ ] No TypeScript errors
- [ ] PR reviewed and approved
- [ ] All acceptance criteria verified
- [ ] Jira story transitioned to Done
```

### Step 6: Jira Sync
**6a. Create sub-tasks** via bulk create:
```
POST $JIRA_BASE_URL/rest/api/3/issue/bulk
{
  "issueUpdates": [
    {
      "fields": {
        "project": { "key": "{PROJECT_KEY}" },
        "summary": "Phase 1: {task title}",
        "issuetype": { "name": "Sub-task" },
        "parent": { "key": "{storyKey}" },
        "description": {
          "type": "doc", "version": 1,
          "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "{task description}" }] }]
        },
        "labels": ["sdlc-plugin", "phase-4-development"]
      }
    }
    // ... one entry per implementation phase
  ]
}
```

**6b. Transition story to In Progress:**
First get available transitions:
```
GET $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/transitions
```
Then transition:
```
POST $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/transitions
{ "transition": { "id": "{in_progress_transition_id}" } }
```

**6c. Add plan summary as a comment:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Implementation plan created. {N} sub-tasks added. Plan file: .claude/SDLCs/plans/plan-{storyKey}.md" }
    ]}]
  }
}
```

## Output
1. **Local file:** `.claude/SDLCs/plans/plan-{storyKey}.md`
2. **Jira actions:**
   - Sub-tasks created under the story (one per implementation phase)
   - Story transitioned to "In Progress"
   - Comment added with plan summary
3. **Summary:** Print a table of created sub-tasks with their keys

## Validation
- [ ] Plan file written to `.claude/SDLCs/plans/plan-{storyKey}.md`
- [ ] All implementation phases have concrete file targets
- [ ] Jira sub-tasks created (keys returned in API response)
- [ ] Story status is now "In Progress"
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-implement plan-{storyKey}.md`
