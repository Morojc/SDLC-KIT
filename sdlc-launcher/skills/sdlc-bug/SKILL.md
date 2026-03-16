---
name: sdlc-bug
description: "SDLC Phase 5: File and triage a bug with reproduction steps"
---

# sdlc-bug — SDLC Phase 5: Testing

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
- `.claude/SDLCs/releases/` — current release context (to determine affected version)
- `CLAUDE.md` (if exists) — project conventions

## Input
$ARGUMENTS — A description of the bug in quotes (e.g. `"Login fails with Google OAuth when redirect URI contains a port number"`)

## Task
Create a structured Jira Bug ticket with reproduction steps, severity assessment, affected version, and a local bug report file.

### Step 1: Parse the Bug Description
From the argument string, extract or infer:
- **Title:** Short, specific, imperative description of the failure (≤ 72 chars)
- **Component:** Which part of the system is affected (infer from description keywords)
- **Severity:** One of: `Critical` / `High` / `Medium` / `Low`
  - Critical: data loss, security vulnerability, system crash, complete feature failure
  - High: major feature broken, no workaround
  - Medium: feature degraded, workaround exists
  - Low: cosmetic, minor inconvenience
- **Affected version:** Read latest release from `.claude/SDLCs/releases/` or use `main`/`HEAD`

### Step 2: Gather Reproduction Context
Prompt for (or infer from context) the following:
- **Environment:** OS, browser, Node version, deployment environment (local/staging/prod)
- **User role:** Who encounters this bug (admin, end-user, API consumer)
- **Frequency:** Always / Intermittent / Rare

If the bug description contains a stack trace or error message, include it verbatim.

### Step 3: Write Reproduction Steps
Generate clear, numbered reproduction steps in the format:
```
1. Navigate to {URL or screen}
2. {Action taken}
3. {Expected result}
4. {Actual result — the bug}
```

Include any preconditions (logged in as X, feature flag Y enabled, etc.).

### Step 4: Assess Impact
- **User impact:** How many users are affected? What can't they do?
- **Data integrity:** Is any data at risk?
- **Workaround:** Is there a temporary mitigation?
- **Related issues:** Search existing bug reports for similar patterns

### Step 5: Write the Bug Report File
Write to `.claude/SDLCs/test-reports/bug-{generatedKey}.md` (use `bug-{date}-{slug}` if no Jira key yet):

```markdown
# Bug Report: {title}

## Metadata
- **Severity:** {Critical/High/Medium/Low}
- **Priority:** {Urgent/High/Medium/Low}
- **Affected Version:** {version}
- **Environment:** {environment details}
- **Reported:** {YYYY-MM-DD}
- **Reporter:** {user or "automated"}

## Description
{Clear description of what is broken and why it matters}

## Reproduction Steps
1. {step}
2. {step}
3. {step}

**Expected:** {what should happen}
**Actual:** {what actually happens}

## Error Output
```
{Stack trace or error message if available}
```

## Impact
- **Users affected:** {scope}
- **Data integrity risk:** {yes/no — describe}
- **Workaround:** {describe or "None"}

## Root Cause Hypothesis
{Initial hypothesis based on the description — what code path likely fails}

## Suggested Fix
{High-level fix direction if obvious}

## Jira Reference
- Bug Key: {PROJ-XXX}
- Story/Epic: {if known}
- Status: Open
```

### Step 6: Jira Sync
Create a Bug issue in Jira:
```
POST $JIRA_BASE_URL/rest/api/3/issue
Authorization: Basic base64($JIRA_EMAIL:$JIRA_API_TOKEN)
{
  "fields": {
    "project": { "key": "{PROJECT_KEY}" },
    "summary": "{bug title}",
    "issuetype": { "name": "Bug" },
    "priority": { "name": "{priority}" },
    "description": {
      "type": "doc", "version": 1,
      "content": [
        { "type": "heading", "attrs": { "level": 3 }, "content": [{ "type": "text", "text": "Steps to Reproduce" }] },
        { "type": "orderedList", "content": [
          { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "{step 1}" }] }] }
          // ... one listItem per step
        ]},
        { "type": "heading", "attrs": { "level": 3 }, "content": [{ "type": "text", "text": "Expected vs Actual" }] },
        { "type": "paragraph", "content": [{ "type": "text", "text": "Expected: {expected}\nActual: {actual}" }] },
        { "type": "heading", "attrs": { "level": 3 }, "content": [{ "type": "text", "text": "Environment" }] },
        { "type": "paragraph", "content": [{ "type": "text", "text": "{environment details}" }] }
      ]
    },
    "labels": ["sdlc-plugin", "phase-5-testing", "bug"],
    "versions": [{ "name": "{affected version}" }]
  }
}
```

After creation, update the bug report file with the returned Jira key and rename the file to `bug-{JIRA-KEY}.md`.

## Output
1. **Local file:** `.claude/SDLCs/test-reports/bug-{JIRA-KEY}.md`
2. **Jira action:** Bug ticket created with full description, reproduction steps, severity, and affected version
3. **Summary:** Bug key, title, severity, and link to Jira ticket

## Validation
- [ ] Bug file written to `.claude/SDLCs/test-reports/bug-{JIRA-KEY}.md`
- [ ] Jira bug ticket created with correct `issuetype: Bug`
- [ ] Reproduction steps are numbered and specific (not vague)
- [ ] Severity and priority fields set
- [ ] Affected version populated
- [ ] Bug key printed for follow-up

## Next Step
After completion, suggest: `/sdlc-qa {storyKey}` (to run full QA and catch related issues) or `/sdlc-hotfix {JIRA-KEY}` if severity is Critical.
