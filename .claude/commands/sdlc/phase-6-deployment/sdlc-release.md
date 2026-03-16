# sdlc-release — SDLC Phase 6: Deployment

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings (base URL, project key, auth)
- `.claude/SDLCs/epics/` — epic artifacts for the target epic
- `.claude/SDLCs/test-reports/` — QA results and test coverage reports
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — `[version] [epic-key]`

- `version` — semantic version string (e.g., `v1.2.0`)
- `epic-key` — Jira Epic key whose completed stories form this release (e.g., `PROJ-002`)

## Task
Generate a release document and create the corresponding Jira Release (Version) for the given epic.

### Step 1: Query Completed Stories
Use JQL to find all stories in the epic that are ready for release:
```
project = {PROJECT_KEY} AND "Epic Link" = {epic-key} AND status = Done ORDER BY priority ASC
```
Also query for bug fixes:
```
project = {PROJECT_KEY} AND "Epic Link" = {epic-key} AND issuetype = Bug AND status = Done
```

### Step 2: Generate Release Notes
From the story results, categorize into:
- **Features** — new Story-type issues
- **Bug Fixes** — Bug-type issues
- **Improvements** — Task-type improvements

Format each entry as: `- [{ISSUE-KEY}] {summary}`

### Step 3: Build Deployment Checklist
Include standard pre-deployment items:
- [ ] All CI checks pass
- [ ] Staging deployment verified
- [ ] Database migrations run and tested
- [ ] Feature flags configured
- [ ] Rollback plan documented
- [ ] Performance benchmarks within baseline
- [ ] Security scan completed
- [ ] Stakeholders notified
- [ ] Monitoring alerts active
- [ ] On-call rotation briefed

### Step 4: Write Release Document
Save to `.claude/SDLCs/releases/release-{version}.md` using this schema:

```markdown
# Release: v{X.X.X}
## Date: {YYYY-MM-DD}
## Jira Release Key: {PROJ-VERSION}

## What's New
### Features
- [{PROJ-XXX}] Feature description

### Bug Fixes
- [{PROJ-YYY}] Bug description

### Improvements
- [{PROJ-ZZZ}] Improvement description

## Breaking Changes
{List any breaking changes with migration guide, or "None"}

## Deployment Checklist
- [ ] All CI checks pass
- [ ] Staging deployment verified
- [ ] Database migrations run
- [ ] Feature flags configured
- [ ] Rollback plan documented
- [ ] Stakeholders notified
- [ ] Monitoring alerts active

## Rollback Plan
{Steps to revert if issues found}

## Known Issues
{Any known limitations in this release, or "None"}
```

### Step 5: Jira Sync — Create Release Version
Execute the following Jira API call:

```
POST {JIRA_BASE_URL}/rest/api/3/version
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "name": "{version}",
  "description": "Release {version} — {epic summary}",
  "projectId": "{project_id}",
  "released": false,
  "releaseDate": "{YYYY-MM-DD}"
}
```

Then add the release notes as a comment on the Epic:
```
POST {JIRA_BASE_URL}/rest/api/3/issue/{epic-key}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Release {version} notes attached. See .claude/SDLCs/releases/release-{version}.md" }
    ]}]
  }
}
```

## Output
1. **Local file:** `.claude/SDLCs/releases/release-{version}.md`
2. **Jira action:** Create Version via `POST /rest/api/3/version`; add release notes comment on Epic
3. **Summary:** Confirm version created, number of stories included, release doc path

## Validation
- [ ] Release document written to `.claude/SDLCs/releases/release-{version}.md`
- [ ] Jira Release (Version) created with correct name and date
- [ ] Release notes comment added to Epic issue
- [ ] All included stories verified as Done status
- [ ] Next command printed

## Next Step
After completion, suggest: `/sdlc-deploy {version} staging`
