# sdlc-hotfix — SDLC Phase 7: Maintenance

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/releases/` — recent release documents for context
- `CLAUDE.md` (if exists) — project branching conventions and emergency procedures

## Input
$ARGUMENTS — `[bug-key]`

- `bug-key` — Jira Bug ticket key for the critical issue requiring a hotfix (e.g., `PROJ-123`)

## Task
Initiate an expedited hotfix workflow: generate a branch strategy, expedited review checklist, and emergency deploy procedure. Link everything back to the original bug ticket.

### Step 1: Load Bug Context
Fetch the bug details from Jira:
```
GET {JIRA_BASE_URL}/rest/api/3/issue/{bug-key}
```

Extract:
- Bug summary and description
- Severity / priority
- Affected version (fixVersion)
- Reporter and assignee
- Related epic or release

### Step 2: Generate Hotfix Branch Strategy
Based on the bug context, define the branch strategy:

**Branch naming convention:**
```
hotfix/{bug-key}-{short-description}
e.g., hotfix/PROJ-123-fix-payment-null-pointer
```

**Branch from:** The production release tag or main/master
```bash
git checkout -b hotfix/{bug-key}-{short-description} {release-tag or main}
```

**Merge targets:**
1. `main` (or `master`) — primary production branch
2. `develop` — keep development branch in sync

Provide the exact git commands:
```bash
# 1. Create hotfix branch
git checkout -b hotfix/{bug-key}-{description} {release-tag}

# 2. After fix is implemented and tested:
git add .
git commit -m "fix({bug-key}): {brief description of fix}"

# 3. Merge to main
git checkout main
git merge --no-ff hotfix/{bug-key}-{description} -m "hotfix: merge {bug-key} to main"
git tag -a v{hotfix-version} -m "Hotfix release {hotfix-version}"

# 4. Merge to develop
git checkout develop
git merge --no-ff hotfix/{bug-key}-{description}
```

### Step 3: Expedited Review Checklist
Due to the emergency nature, apply a compressed but rigorous review process:

**Root Cause Analysis**
- [ ] Root cause identified and documented
- [ ] Fix scope limited to the minimum necessary change
- [ ] No unrelated changes included

**Code Review (expedited — 2 reviewers minimum)**
- [ ] Primary reviewer: code correctness verified
- [ ] Secondary reviewer: security implications checked
- [ ] No new bugs introduced by the fix
- [ ] Edge cases considered

**Testing (expedited)**
- [ ] Regression test for the reported bug written
- [ ] Affected unit tests pass
- [ ] Smoke test on staging environment
- [ ] Integration tests pass for affected area

**Pre-Deploy**
- [ ] Rollback plan from this hotfix documented
- [ ] Database changes (if any) are backward compatible
- [ ] Feature flags available for instant disable if needed
- [ ] On-call engineer briefed

### Step 4: Emergency Deploy Procedure
```
1. Merge hotfix branch to main (after review approval)
2. Tag the hotfix release: v{major}.{minor}.{patch+1}
3. Trigger CI/CD pipeline for emergency deploy
4. Monitor for 30 minutes post-deploy
5. Notify stakeholders of resolution
6. Merge hotfix branch back to develop
```

### Step 5: Write Hotfix Plan Document
Generate summary in chat (no separate file needed unless the fix is complex):
- Bug: {bug-key} — {summary}
- Hotfix branch: hotfix/{bug-key}-{description}
- Hotfix version: v{X.X.Y}
- Root cause: {1-2 sentences}
- Fix approach: {1-2 sentences}
- Emergency contacts: {from jira-config or CLAUDE.md}

### Step 6: Jira Sync
Transition the bug ticket to In Progress:
```
POST {JIRA_BASE_URL}/rest/api/3/issue/{bug-key}/transitions
{
  "transition": { "id": "{transition_id_for_in_progress}" }
}
```

Add a comment with the hotfix plan:
```
POST {JIRA_BASE_URL}/rest/api/3/issue/{bug-key}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Hotfix initiated. Branch: hotfix/{bug-key}-{description}. Target release: v{X.X.Y}. Estimated resolution: {timeframe}." }
    ]}]
  }
}
```

Link hotfix to the release that introduced the bug:
```
POST {JIRA_BASE_URL}/rest/api/3/issueLink
{
  "type": { "name": "Relates" },
  "inwardIssue": { "key": "{bug-key}" },
  "outwardIssue": { "key": "{affected-release-epic-key}" }
}
```

## Output
1. **Local file:** None (hotfix summary output in chat; complex fixes may warrant a brief doc)
2. **Jira action:** Transition bug to In Progress; add hotfix plan comment; link to release
3. **Summary:** Hotfix plan ready — branch name, version target, expedited review checklist

## Validation
- [ ] Root cause documented
- [ ] Hotfix branch name and git commands provided
- [ ] Expedited review checklist complete
- [ ] Emergency deploy procedure outlined
- [ ] Bug ticket transitioned to In Progress in Jira
- [ ] Hotfix comment added to bug ticket
- [ ] Next command printed

## Next Step
After fix is implemented and merged:
`/sdlc-deploy v{hotfix-version} production`
