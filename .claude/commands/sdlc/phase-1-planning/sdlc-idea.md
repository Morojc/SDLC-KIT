# sdlc-idea — SDLC Phase 1: Planning

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings (base URL, project key, auth)
- `.claude/SDLCs/ideas/` — existing idea documents for numbering continuity
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — A raw idea description in natural language (e.g., "Build a real-time collaboration feature for documents")

## Task
Refine the raw idea into a structured concept document with problem statement, target users, and success metrics. Then create a Jira Initiative ticket.

### Step 1: Parse the Idea
Analyze $ARGUMENTS and extract:
- Core problem being solved
- Target user segment(s)
- Proposed high-level solution
- Measurable success metrics (at least 2)
- Risks and assumptions (at least 2 of each)

### Step 2: Determine Next Idea Number
List files in `.claude/SDLCs/ideas/` and determine the next sequential number (e.g., `idea-001.md`, `idea-002.md`).

### Step 3: Write the Idea Document
Create `.claude/SDLCs/ideas/idea-XXX.md` using this schema:

```markdown
# Idea: [Title derived from input]

## Problem Statement
[What problem does this solve? Be specific about pain points.]

## Target Users
[Who benefits? Include user roles, scale, frequency of use.]

## Proposed Solution
[High-level solution — no implementation details yet. What changes for users?]

## Success Metrics
- Metric 1: [measurable outcome with target, e.g., "Reduce onboarding time by 40%"]
- Metric 2: [measurable outcome with target]

## Risks & Assumptions
- Risk 1: [description + likelihood]
- Risk 2: [description + likelihood]
- Assumption 1: [what must be true for this to work]
- Assumption 2: [what must be true for this to work]

## Jira Reference
- Initiative Key: [PROJ-XXX — fill after Jira creation]
- Created: [today's date]
- Status: To Do
```

### Step 4: Jira Sync — Create Initiative
Create a Jira Initiative (or Epic if Initiatives not available) via the REST API.

Read `JIRA_BASE_URL`, `JIRA_EMAIL`, `JIRA_API_TOKEN`, and `JIRA_PROJECT_KEY` from environment or `.claude/settings/jira-config.json`.

Provide the user with this curl command to execute:

```bash
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "$JIRA_PROJECT_KEY" },
      "summary": "[Title of idea]",
      "description": {
        "type": "doc", "version": 1,
        "content": [{
          "type": "paragraph",
          "content": [{ "type": "text", "text": "[Problem statement + proposed solution]" }]
        }]
      },
      "issuetype": { "name": "Initiative" },
      "priority": { "name": "Medium" },
      "labels": ["sdlc-plugin", "phase-1-planning", "idea"]
    }
  }'
```

After the user runs the command, update the `Jira Reference` section of the idea file with the returned issue key.

## Output
1. **Local file:** `.claude/SDLCs/ideas/idea-XXX.md`
2. **Jira action:** Create Initiative with summary = idea title, description = problem + solution
3. **Summary:** Print the idea title, file path, and Jira key (once created)

## Validation
Before finishing, confirm:
- [ ] Idea file written to `.claude/SDLCs/ideas/idea-XXX.md`
- [ ] All sections populated (problem, users, solution, metrics, risks)
- [ ] Jira curl command provided to user
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-epic idea-XXX.md`
