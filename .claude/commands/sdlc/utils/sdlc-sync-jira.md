# sdlc-sync-jira — SDLC Utility: Sync Artifact to Jira

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- The artifact file specified in $ARGUMENTS
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — `[artifact-file]`

- `artifact-file` — path to a local `.claude/SDLCs/` artifact file to push to Jira
  - Examples:
    - `.claude/SDLCs/prds/prd-PROJ-002.md`
    - `.claude/SDLCs/plans/plan-PROJ-010.md`
    - `.claude/SDLCs/designs/design-PROJ-010.md`
    - `.claude/SDLCs/releases/release-v1.2.0.md`
    - `.claude/SDLCs/retros/retro-sprint-1.md`

## Task
Read the specified local artifact, detect its type, determine the correct Jira target, and push the content as a description update or comment.

### Step 1: Read and Classify the Artifact
Read the artifact file. Determine its type by examining:
- File path prefix (e.g., `prds/`, `plans/`, `designs/`, `releases/`, `retros/`)
- File name pattern (e.g., `prd-PROJ-XXX.md`, `plan-PROJ-XXX.md`)
- First heading in the file content

**Artifact Type → Jira Target Mapping:**

| Artifact Type | Identified By | Jira Target | Action |
|---------------|---------------|-------------|--------|
| Idea | `ideas/idea-*.md` | Initiative issue | Update description |
| PRD | `prds/prd-{KEY}.md` | Epic {KEY} | Update description + add comment |
| Epic | `epics/epic-*.md` | Epic issue | Update description |
| Design | `designs/design-{KEY}.md` | Story {KEY} | Add comment |
| Implementation Plan | `plans/plan-{KEY}.md` | Story {KEY} | Add comment + create sub-tasks |
| Test Report | `test-reports/` | Story or Epic | Add comment |
| Release Notes | `releases/release-{version}.md` | Version + Epic | Update version description |
| Retrospective | `retros/retro-*.md` | Sprint or Epic | Add comment |

### Step 2: Extract Jira Issue Key
Parse the Jira issue key from:
1. The file name (e.g., `plan-PROJ-010.md` → `PROJ-010`)
2. The "Jira Reference" section in the artifact content
3. The `## Jira Context` section (for plans)

Fallback: prompt the user if the key cannot be determined automatically.

### Step 3: Convert Markdown to Jira Document Format
Convert the artifact markdown to Jira's Atlassian Document Format (ADF):

For simple content, wrap in ADF paragraph nodes:
```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "{content}" }
      ]
    }
  ]
}
```

For structured content, convert markdown headings → ADF heading nodes, lists → ADF bulletList nodes, code blocks → ADF codeBlock nodes.

### Step 4: Push to Jira

**Update Epic/Story Description:**
```
PUT {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "fields": {
    "description": {ADF document}
  }
}
```

**Add as Comment (preferred for plans, designs, retros):**
```
POST {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/comment
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "body": {ADF document}
}
```

**For Release artifacts — update Version description:**
```
GET {JIRA_BASE_URL}/rest/api/3/project/{PROJECT_KEY}/versions
(find the version by name)

PUT {JIRA_BASE_URL}/rest/api/3/version/{versionId}
{
  "description": "{release summary text}"
}
```

### Step 5: Confirm Sync
Report:
- Artifact file: {path}
- Artifact type: {detected type}
- Pushed to: {Jira issue key / version}
- Action taken: {description update / comment added}
- Jira URL: {JIRA_BASE_URL}/browse/{issueKey}

## Output
1. **Local file:** None (artifact file is read-only; no local changes)
2. **Jira action:** Update description or add comment on the target Jira issue
3. **Summary:** Sync confirmed — artifact type, target issue key, action taken

## Validation
- [ ] Artifact file read successfully
- [ ] Artifact type correctly detected
- [ ] Jira issue key extracted
- [ ] Content pushed to correct Jira field (description or comment)
- [ ] Jira URL printed for verification

## Next Step
After syncing, verify the result in Jira:
`open {JIRA_BASE_URL}/browse/{issueKey}`
