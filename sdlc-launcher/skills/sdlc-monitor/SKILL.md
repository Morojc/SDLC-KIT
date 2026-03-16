---
name: sdlc-monitor
description: "SDLC Phase 7: Check production health and create incident tickets"
---

# sdlc-monitor — SDLC Phase 7: Maintenance

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
- `.claude/SDLCs/releases/release-{release-key}.md` — release document and deployment notes
- `CLAUDE.md` (if exists) — project conventions and monitoring configuration

## Input
$ARGUMENTS — `[release-key]`

- `release-key` — version/release identifier to monitor (e.g., `v1.2.0`)

## Task
Run a post-deployment monitoring checklist, summarize system health, and create Jira Bug tickets for any issues discovered.

### Step 1: Load Release Context
Read `.claude/SDLCs/releases/release-{release-key}.md` to understand:
- What was deployed (features, bug fixes, changes)
- Known issues at release time
- Rollback plan (for reference if issues detected)
- Deployment date and environment

### Step 2: Run Monitoring Checklist
Work through each monitoring category. For each item, assess status as ✅ OK, ⚠️ Warning, or ❌ Critical:

**Error Rates & Availability**
- [ ] Application error rate within baseline (< 0.1% for production)
- [ ] HTTP 5xx error rate below threshold
- [ ] Service availability / uptime confirmed (> 99.9%)
- [ ] Dependency health (databases, queues, external APIs)

**Performance**
- [ ] API response times within SLA (e.g., p95 < 500ms)
- [ ] Database query performance unchanged or improved
- [ ] Memory usage stable (no leak indicators)
- [ ] CPU utilization within normal range
- [ ] CDN and cache hit rates healthy

**User Experience**
- [ ] Core user flows tested end-to-end
- [ ] Frontend error rate in acceptable range
- [ ] Mobile client error rates monitored
- [ ] Accessibility compliance maintained

**Data Integrity**
- [ ] Database row counts align with expected post-migration values
- [ ] No data corruption detected
- [ ] Backup jobs running successfully
- [ ] Audit logs capturing expected events

**Security**
- [ ] No new authentication failures or suspicious login patterns
- [ ] Rate limiting functioning correctly
- [ ] Security headers present and correct
- [ ] No exposed sensitive data in logs

**Business Metrics** (24-48 hours post-deploy)
- [ ] Key conversion metrics not degraded
- [ ] Feature adoption tracking correctly
- [ ] User feedback channels monitored

### Step 3: Summarize Findings
Produce a monitoring report with:
- Overall health status: 🟢 Healthy / 🟡 Degraded / 🔴 Critical
- Issues found (if any)
- Recommended actions

### Step 4: Create Jira Bug Tickets for Issues
For each ⚠️ Warning or ❌ Critical issue found, create a Jira Bug:

```
POST {JIRA_BASE_URL}/rest/api/3/issue
Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}
Content-Type: application/json

{
  "fields": {
    "project": { "key": "{PROJECT_KEY}" },
    "summary": "[{release-key}] {Issue description}",
    "description": {
      "type": "doc", "version": 1,
      "content": [{ "type": "paragraph", "content": [
        { "type": "text", "text": "Post-deployment monitoring issue detected for release {release-key}.\n\nSymptom: {description}\nSeverity: {Critical|High|Medium}\nDetected at: {timestamp}\nAffected component: {component}" }
      ]}]
    },
    "issuetype": { "name": "Bug" },
    "priority": { "name": "{Critical|High|Medium}" },
    "labels": ["sdlc-plugin", "post-deploy-monitoring", "{release-key}"]
  }
}
```

Link each bug to the release epic:
```
POST {JIRA_BASE_URL}/rest/api/3/issueLink
{
  "type": { "name": "Relates" },
  "inwardIssue": { "key": "{bug-key}" },
  "outwardIssue": { "key": "{epic-key}" }
}
```

### Step 5: Update Release Document
Append monitoring results to `.claude/SDLCs/releases/release-{release-key}.md`:
- Monitoring start timestamp
- Overall health status
- List of bugs created with Jira keys
- Next monitoring checkpoint

## Output
1. **Local file:** Appends monitoring report to `.claude/SDLCs/releases/release-{release-key}.md`
2. **Jira action:** Create Bug tickets for each issue found via `POST /rest/api/3/issue`
3. **Summary:** Monitoring complete — overall status, N issues found, N Jira bugs created

## Validation
- [ ] All monitoring checklist items assessed
- [ ] Release document updated with monitoring findings
- [ ] Jira Bug tickets created for all ⚠️/❌ items
- [ ] Bug tickets linked to parent Epic
- [ ] Next command printed

## Next Step
After completion:
- If critical issues found: `/sdlc-hotfix {bug-key}`
- If healthy: `/sdlc-retro {release-key}`
