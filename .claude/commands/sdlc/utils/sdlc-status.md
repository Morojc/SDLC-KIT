# sdlc-status — SDLC Utility: Project Dashboard

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/` — all artifact directories for local context
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — `[epic-key or sprint-id]` (optional)

- `epic-key` — Jira Epic key to scope the dashboard (e.g., `PROJ-002`)
- `sprint-id` — Sprint name or ID to show sprint-level status
- If omitted, shows the overall project status across all active epics

## Task
Display a comprehensive SDLC dashboard showing current phase, open tickets, blockers, and the recommended next action.

### Step 1: Query Jira for Current State

**If epic-key provided:**
```
GET {JIRA_BASE_URL}/rest/api/3/search?jql=project={PROJECT_KEY} AND "Epic Link"={epic-key} ORDER BY status ASC
GET {JIRA_BASE_URL}/rest/api/3/issue/{epic-key}
```

**If sprint-id provided:**
```
GET {JIRA_BASE_URL}/rest/agile/1.0/sprint/{sprintId}/issue
```

**If no argument (full project view):**
```
GET {JIRA_BASE_URL}/rest/api/3/search?jql=project={PROJECT_KEY} AND issuetype=Epic AND status != Done ORDER BY created DESC
GET {JIRA_BASE_URL}/rest/api/3/search?jql=project={PROJECT_KEY} AND priority=Blocker AND status != Done
```

### Step 2: Determine Current SDLC Phase
Analyze the state of issues to infer the current phase:

| If... | Then current phase is... |
|-------|--------------------------|
| All stories in To Do, no designs | Phase 2/3 — Requirements/Design |
| Stories in To Do, designs exist | Phase 4 — Development |
| Stories In Progress | Phase 4 — Development |
| Stories In Review | Phase 5 — Testing/QA |
| Stories Done, no release | Phase 6 — Deployment (Release pending) |
| Release created, not deployed | Phase 6 — Deployment |
| Release deployed, no retro | Phase 7 — Maintenance |

### Step 3: Compile Status Metrics
Gather:
- Total stories: planned / in-progress / in-review / done
- Story points: committed / completed / remaining
- Bug count: open / resolved
- Blockers: count and list
- Sprint velocity (if sprint context available)
- Days until sprint end (if applicable)

### Step 4: Identify Blockers
Query for all blocking issues:
```
project={PROJECT_KEY} AND priority=Blocker AND status != Done
project={PROJECT_KEY} AND issueType in (Bug) AND priority in (Critical, Blocker) AND status != Done
```

### Step 5: Render Dashboard
Output a formatted status dashboard:

```
╔══════════════════════════════════════════════════════════════╗
║  SDLC STATUS DASHBOARD                                        ║
║  Project: {PROJECT_KEY} | {Epic/Sprint Name}                 ║
║  As of: {timestamp}                                           ║
╚══════════════════════════════════════════════════════════════╝

📍 CURRENT PHASE: Phase {N} — {Phase Name}

📊 PROGRESS
  Stories:      {done}/{total} complete ({percent}%)
  Story Points: {completed}/{committed} ({remaining} remaining)
  Bugs Open:    {open_bugs}
  Bugs Resolved:{resolved_bugs}

🚨 BLOCKERS ({blocker_count})
  {PROJ-XXX} — {blocker summary} [assigned to: {assignee}]
  ...

📋 IN PROGRESS
  {PROJ-XXX} — {story summary} [{assignee}] ({points} pts)
  ...

✅ RECENTLY COMPLETED (last 5)
  {PROJ-XXX} — {story summary}
  ...

📅 SPRINT HEALTH (if applicable)
  Sprint: {Sprint Name}
  Days remaining: {N}
  Velocity on track: {Yes/No}

🎯 RECOMMENDED NEXT ACTION
  {Specific command and reasoning based on current state}
  Example: /sdlc-deploy v1.2.0 staging
           (All stories done, release v1.2.0 created, staging not yet deployed)
```

### Step 6: Suggest Next Action
Based on the current phase and state, provide a specific recommended next command with explanation.

## Output
1. **Local file:** None (dashboard output in chat)
2. **Jira action:** Read-only JQL queries (no writes)
3. **Summary:** Current phase, health status, and single most important next action

## Validation
- [ ] Jira queried successfully for current state
- [ ] SDLC phase correctly inferred
- [ ] Blockers identified and listed
- [ ] Recommended next action provided with specific command
- [ ] Dashboard rendered clearly

## Next Step
After reviewing status, follow the recommended next action shown in the dashboard.
