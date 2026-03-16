---
name: sdlc-pull
description: "SDLC Utility: Pull latest Jira changes and reconcile with local artifacts"
---

# sdlc-pull — SDLC Utility: Jira Pull & Reconcile

## MCP Tools
- **Jira** (`jira` MCP): `jira_get_issue`, `jira_search_issues` — fetch latest state from Jira
- **GitHub** (`github` MCP): `get_file_contents`, `push_files` — sync docs if needed

## Mulch & Seeds

### Session Start
```bash
mulch prime --context
sd prime
```

### Session Close
```bash
mulch learn
mulch record sdlc-commands --type convention --description "..."
mulch sync
sd close <id>
sd sync
```

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project key and base URL
- `.claude/SDLCs/` — all local artifact directories (epics, prds, stories, plans, designs)
- `CLAUDE.md` — project conventions

## Input
$ARGUMENTS — Optional. One of:
- A Jira section: `epic`, `stories`, `sprint`, `all`
- A specific Jira key: `PROJ-12`, `PROJ-5`
- Nothing (interactive mode — will ask)

## Task
Pull the latest state from Jira, diff against local artifacts, and update local files to reflect what changed. This is the **Jira → local** sync direction (opposite of `sdlc-sync-jira`).

---

### Step 0: Jira Change Check (Interactive)

If $ARGUMENTS is empty, ask the user:

```
Have you made any manual changes in Jira since your last session? (yes / no)
```

If **no** → exit with "Local artifacts are up to date. Nothing to pull."

If **yes** → ask:

```
Which section did you update?
  1. Epic
  2. Stories (all)
  3. Specific ticket (enter key, e.g. PROJ-12)
  4. Sprint
  5. Everything
```

Proceed based on their answer. If $ARGUMENTS was provided, skip the questions and use it directly.

---

### Step 1: Fetch Latest from Jira

Based on the selected scope:

**Epic:**
```
# MCP: jira_get_issue
# Fetch the epic — summary, description, status, acceptance criteria
```

**Stories (all):**
```
# MCP: jira_search_issues
# JQL: project={PROJECT_KEY} AND issuetype=Story AND "Epic Link"={epicKey}
# Fetch: summary, status, description, acceptance criteria, story points, assignee
```

**Specific ticket:**
```
# MCP: jira_get_issue
# Fetch: all fields for the given key
```

**Sprint:**
```
# MCP: jira_search_issues
# JQL: project={PROJECT_KEY} AND sprint in openSprints()
# Fetch: sprint name, goal, stories, statuses
```

**Everything:**
Run epic + stories + sprint fetch in sequence.

---

### Step 2: Read Local Artifacts

For each fetched Jira item, find the corresponding local file:

| Jira item | Local file |
|-----------|-----------|
| Epic | `.claude/SDLCs/epics/epic-{key}.md` |
| Story | `.claude/SDLCs/stories/story-{key}.md` |
| PRD | `.claude/SDLCs/prds/prd-{epicKey}.md` |
| Sprint plan | `.claude/SDLCs/plans/sprint-{N}-plan.md` |
| Design | `.claude/SDLCs/designs/design-{key}.md` |

If no local file exists for a Jira item → create it from Jira data.

---

### Step 3: Diff and Report Changes

For each Jira item vs local file, compare:

| Field | Jira value | Local value | Action |
|-------|-----------|-------------|--------|
| Status | In Progress | To Do | Update local |
| Story points | 5 | 3 | Update local |
| Acceptance criteria | (updated) | (old) | Update local |
| Assignee | @alice | unassigned | Update local |
| Summary | (renamed) | (old name) | Update local + rename file |

Print a diff table:
```
JIRA PULL REPORT
────────────────────────────────────────
✓ MA-3  status: To Do → In Progress
✓ MA-5  points: 2 → 3
✓ MA-7  acceptance criteria: updated (3 lines changed)
  MA-9  no changes
✗ MA-6  local file missing → created from Jira
────────────────────────────────────────
4 items checked | 3 updated | 1 created | 1 unchanged
```

Ask for confirmation before writing:
```
Apply these changes to local artifacts? (yes / no / show-diff)
```

---

### Step 4: Update Local Files

For each changed item, update the local markdown file:
- Overwrite changed fields only (preserve local-only sections like "Technical Notes", "Implementation Notes")
- Add a `## Last Pulled` frontmatter field with the current timestamp
- Do NOT overwrite sections that exist only locally (e.g., `## Implementation Notes`, `## File Targets`)

---

### Step 5: Record Reconciliation

After all updates, record what changed in Seeds and Mulch:

```bash
# Seeds: update issue status to match Jira
sd update <id> --status in_progress  # for any story now In Progress in Jira

# Mulch: record if any architectural decisions changed
mulch record sdlc-commands --type decision \
  --description "Jira pull reconciled: {summary of changes}"
```

---

## Output
1. **Updated local files** — all changed `.claude/SDLCs/` artifacts reconciled with Jira
2. **Pull report** — printed diff table showing what changed
3. **No Jira writes** — this is read-only from Jira's perspective

## Validation
- [ ] All fetched Jira items have a corresponding local file (created if missing)
- [ ] Changed fields updated in local files; local-only sections preserved
- [ ] Pull report printed with counts
- [ ] Seeds statuses updated to match Jira

## Next Step
After pulling, continue with your planned work:
- `/sdlc-plan {storyKey}` — plan a story that just moved to In Progress
- `/sdlc-implement {storyKey}` — implement a story now ready
- `/sdlc-kickoff` — start a new project
