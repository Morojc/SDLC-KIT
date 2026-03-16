---
name: sdlc-deploy
description: "SDLC Phase 6: Execute deployment checklist and Jira sync"
---

# sdlc-deploy — SDLC Phase 6: Deployment

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
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/releases/release-{version}.md` — release document for the target version
- `CLAUDE.md` (if exists) — project conventions and deployment notes

## Input
$ARGUMENTS — `[release-key] [environment]`

- `release-key` — version string or Jira release name (e.g., `v1.2.0`)
- `environment` — target deployment environment: `staging`, `production`

## Task
Execute deployment validation, transition all stories in the release to Done, and mark the Jira Release as released.

### Step 1: Load Release Context
Read `.claude/SDLCs/releases/release-{release-key}.md` to get:
- List of included stories/issues
- Deployment checklist status
- Rollback plan
- Known issues

### Step 2: Pre-Deployment Validation
Verify each item in the deployment checklist. For each item, prompt the user to confirm or auto-check if verifiable:
- [ ] All CI pipeline checks pass (no failing builds)
- [ ] Staging smoke tests completed (if deploying to production)
- [ ] Database migration scripts reviewed and ready
- [ ] Feature flags set to correct state for this environment
- [ ] Rollback procedure documented and tested
- [ ] Infrastructure capacity confirmed for expected load
- [ ] SSL certificates valid and not expiring within 30 days
- [ ] Third-party service dependencies healthy
- [ ] Monitoring dashboards baseline captured
- [ ] Incident response team on standby (for production)

### Step 3: Environment-Specific Checks
**For staging:**
- Verify staging environment matches production configuration
- Run integration test suite against staging
- Confirm data seeding / migration applied

**For production:**
- Confirm all staging validations passed
- Verify backup of production database taken
- Confirm change management approval (if required)
- Notify stakeholders via established channels

### Step 4: Transition Stories to Done
Query all stories in the release:
```
project = {PROJECT_KEY} AND fixVersion = "{release-key}" AND status != Done
```

For each story found, transition to Done:
```
POST {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
{
  "transition": { "id": "{transition_id_for_done}" }
}
```

First get available transitions:
```
GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
```

### Step 5: Mark Jira Release as Released (production only)
```
PUT {JIRA_BASE_URL}/rest/api/3/version/{versionId}
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "released": true,
  "releaseDate": "{YYYY-MM-DD}"
}
```

### Step 6: Post-Deployment Report
Generate a deployment summary and add as a comment to the Epic:
- Environment deployed to
- Time of deployment
- Stories transitioned to Done
- Any issues encountered
- Next step: monitoring

Update the release document to record deployment outcome.

## Output
1. **Local file:** Updates `.claude/SDLCs/releases/release-{release-key}.md` with deployment status
2. **Jira action:** Transition all release stories to Done; mark Version as released (production)
3. **Summary:** Deployment confirmation with environment, story count, and timestamp

## Validation
- [ ] All pre-deployment checklist items confirmed
- [ ] All included stories transitioned to Done in Jira
- [ ] Jira Release Version marked as released (production deployments)
- [ ] Deployment summary added as comment to Epic
- [ ] Release document updated with deployment outcome
- [ ] Next command printed

## Next Step
After completion, suggest: `/sdlc-monitor {release-key}`
