---
name: sdlc-release
description: "SDLC Phase 6: Create release notes and version tag"
---

# sdlc-release — SDLC Phase 6: Deployment

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

## Git Release Gate

Before tagging a release, verify the sprint → main merge has been completed:

```
# MCP: github — list_pull_requests
# base: main, state: all
# Find the sprint/{N} PR and check it is merged
```

If sprint PR is **not yet merged**:
```
✗ RELEASE BLOCKED — sprint/{N}-{slug} has not been merged to main yet.

Run first: /sdlc-merge sprint/{N}
  → checks all stories are Done
  → opens PR: sprint/{N} → main
  → you merge the PR in GitHub

Then re-run /sdlc-release to tag the version.
```

If sprint PR **is merged** → proceed to create the version tag:

```
# MCP: github — create_tag (or create_release)
# tag:  v{version}
# target: main
# name: "v{version} — {sprint goal}"
# body: {release notes from release doc}
```

## Output
1. **Local file:** `.claude/SDLCs/releases/release-{version}.md`
2. **Jira action:** Jira Version created; release notes comment on Epic
3. **GitHub action:** Version tag `v{version}` created on `main`
4. **Summary:** Version tagged, stories included, release doc path

## Validation
- [ ] Sprint branch merged to `main` before release is tagged
- [ ] Release document written to `.claude/SDLCs/releases/release-{version}.md`
- [ ] Jira Version created with correct name and release date
- [ ] Release notes comment added to Epic
- [ ] All included stories verified as Done status
- [ ] GitHub tag `v{version}` created on `main`

## Next Step
`/sdlc-deploy {version} staging`
