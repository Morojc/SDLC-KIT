# sdlc-epic — SDLC Phase 1: Planning

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/ideas/` — idea documents
- `.claude/SDLCs/epics/` — existing epics for numbering continuity
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — Either:
- A path to an idea file (e.g., `idea-001.md` or `.claude/SDLCs/ideas/idea-001.md`)
- A Jira Initiative key (e.g., `PROJ-001`)

## Task
Convert an idea into a full Jira Epic with description, business value, and acceptance criteria. Write the epic artifact locally and create the Jira Epic ticket linked to the Initiative.

### Step 1: Load Source Material
If $ARGUMENTS is a file path, read the idea document from `.claude/SDLCs/ideas/`.
If $ARGUMENTS is a Jira key, provide a curl to fetch the issue:

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/$ARGUMENTS" | jq '.fields | {summary, description}'
```

### Step 2: Expand into Epic
Using the idea content, produce:
- A clear epic title (action-oriented, e.g., "Implement Real-Time Document Collaboration")
- Business value statement (why this matters to the organization)
- Scope definition (what's in and explicitly what's out)
- High-level acceptance criteria (3–5 measurable conditions)
- Estimated complexity (S/M/L/XL with rationale)
- Dependencies on other systems or teams

### Step 3: Write Epic Document
Create `.claude/SDLCs/epics/epic-XXX.md`:

```markdown
# Epic: [Epic Title]

## Jira Reference
- Epic Key: [PROJ-XXX — fill after creation]
- Initiative Key: [Parent initiative key]
- Created: [date]
- Status: To Do
- Estimated Size: [S/M/L/XL]

## Business Value
[Why does this matter? What business outcome does it drive?]

## Scope

### In Scope
- [Item 1]
- [Item 2]

### Out of Scope
- [Item 1]
- [Item 2]

## Acceptance Criteria
- [ ] [Measurable condition 1]
- [ ] [Measurable condition 2]
- [ ] [Measurable condition 3]

## Dependencies
- [System or team dependency 1]
- [System or team dependency 2]

## Stories Overview
[To be populated by /sdlc-stories]

## Notes
[Any additional context or open questions]
```

### Step 4: Jira Sync — Create Epic
Create the Epic in Jira and link it to the Initiative.

```bash
# Step 4a: Create the Epic
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "$JIRA_PROJECT_KEY" },
      "summary": "[Epic title]",
      "description": {
        "type": "doc", "version": 1,
        "content": [{
          "type": "paragraph",
          "content": [{ "type": "text", "text": "[Business value + scope summary]" }]
        }]
      },
      "issuetype": { "name": "Epic" },
      "customfield_10014": "[Epic title as epic name field]",
      "priority": { "name": "High" },
      "labels": ["sdlc-plugin", "phase-1-planning"]
    }
  }'

# Step 4b: Link Epic to Initiative (replace EPIC-KEY and INITIATIVE-KEY)
curl -X POST \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issueLink" \
  -d '{
    "type": { "name": "Initiative" },
    "inwardIssue": { "key": "[EPIC-KEY]" },
    "outwardIssue": { "key": "[INITIATIVE-KEY]" }
  }'
```

Update the epic file's `Jira Reference` with the returned epic key.

## Output
1. **Local file:** `.claude/SDLCs/epics/epic-XXX.md`
2. **Jira action:** Create Epic linked to Initiative; set epic name custom field
3. **Summary:** Print epic title, file path, and Jira key

## Validation
Before finishing, confirm:
- [ ] Epic file written to `.claude/SDLCs/epics/epic-XXX.md`
- [ ] Business value and scope clearly defined
- [ ] Acceptance criteria are measurable (not vague)
- [ ] Jira Epic created and linked to Initiative
- [ ] Next recommended command printed

## Next Step
After completion, suggest: `/sdlc-roadmap [EPIC-KEY] 4`
