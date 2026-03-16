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

### Step 6: Generate Sprint Report (XLSX)

After tagging the release, generate a well-formatted Excel report for the sprint.

**6a. Collect data:**
- All stories in the sprint (from Jira + local `.claude/SDLCs/stories/` files)
- All sub-tasks under each story (from local `.claude/SDLCs/plans/` files)
- Sprint metadata: name, dates, capacity, velocity

**6b. Write and run a Python generation script:**

```python
# .claude/SDLCs/reports/_gen_sprint_report.py
# Auto-generated by sdlc-release — do not edit manually

import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from datetime import datetime

# ── DATA (filled from Jira + local story files) ──────────────────────────────
SPRINT = {
    "name": "{sprint_name}",
    "goal": "{sprint_goal}",
    "start": "{YYYY-MM-DD}",
    "end": "{YYYY-MM-DD}",
    "capacity_pts": {capacity_points},
    "velocity_pts": {completed_points},
    "version": "{version}",
    "epic_key": "{epic_key}",
}

STORIES = [
    # { key, summary, type, priority, points, status, assignee, subtasks: [...] }
    {
        "key": "{PROJ-XXX}",
        "summary": "{story summary}",
        "type": "dev",
        "priority": "Critical",
        "points": 3,
        "status": "Done",
        "assignee": "{name}",
        "subtasks": [
            {"key": "{PROJ-XXX-1}", "summary": "Phase 1: ...", "status": "Done", "hours": 2},
        ]
    },
    # ... one entry per story
]

# ── STYLES ────────────────────────────────────────────────────────────────────
HEADER_FILL   = PatternFill("solid", fgColor="1F3864")   # dark navy
SUBHDR_FILL   = PatternFill("solid", fgColor="2E75B6")   # mid blue
STORY_FILL    = PatternFill("solid", fgColor="D6E4F0")   # light blue
SUBTASK_FILL  = PatternFill("solid", fgColor="F2F7FB")   # very light blue
DONE_FILL     = PatternFill("solid", fgColor="E2EFDA")   # light green
BLOCKED_FILL  = PatternFill("solid", fgColor="FCE4D6")   # light red

def hdr(bold=True, white=True, size=11):
    return Font(bold=bold, color="FFFFFF" if white else "000000", size=size)

def border():
    s = Side(style="thin", color="BFBFBF")
    return Border(left=s, right=s, top=s, bottom=s)

def center():
    return Alignment(horizontal="center", vertical="center", wrap_text=True)

def left():
    return Alignment(horizontal="left", vertical="center", wrap_text=True)

# ── WORKBOOK ──────────────────────────────────────────────────────────────────
wb = openpyxl.Workbook()

# ── SHEET 1: Sprint Overview ──────────────────────────────────────────────────
ws1 = wb.active
ws1.title = "Sprint Overview"

ws1.column_dimensions["A"].width = 22
ws1.column_dimensions["B"].width = 45

title_cell = ws1["A1"]
title_cell.value = f"Sprint Report — {SPRINT['name']} ({SPRINT['version']})"
title_cell.font = Font(bold=True, size=14, color="FFFFFF")
title_cell.fill = HEADER_FILL
title_cell.alignment = center()
ws1.merge_cells("A1:B1")
ws1.row_dimensions[1].height = 30

fields = [
    ("Sprint Name",     SPRINT["name"]),
    ("Sprint Goal",     SPRINT["goal"]),
    ("Start Date",      SPRINT["start"]),
    ("End Date",        SPRINT["end"]),
    ("Epic Key",        SPRINT["epic_key"]),
    ("Release Version", SPRINT["version"]),
    ("Capacity (pts)",  SPRINT["capacity_pts"]),
    ("Velocity (pts)",  SPRINT["velocity_pts"]),
    ("Completion",      f"{round(SPRINT['velocity_pts']/SPRINT['capacity_pts']*100)}%"),
    ("Generated",       datetime.now().strftime("%Y-%m-%d %H:%M")),
]

for i, (label, value) in enumerate(fields, start=2):
    lc = ws1.cell(row=i, column=1, value=label)
    lc.font = Font(bold=True, color="FFFFFF")
    lc.fill = SUBHDR_FILL
    lc.alignment = left()
    lc.border = border()
    vc = ws1.cell(row=i, column=2, value=value)
    vc.alignment = left()
    vc.border = border()

# ── SHEET 2: Stories & Tasks ──────────────────────────────────────────────────
ws2 = wb.create_sheet("Stories & Tasks")

col_widths = [12, 50, 10, 12, 8, 14, 18, 8]
col_headers = ["Key", "Summary", "Type", "Priority", "Points", "Status", "Assignee", "Hours"]
for ci, (w, h) in enumerate(zip(col_widths, col_headers), start=1):
    ws2.column_dimensions[get_column_letter(ci)].width = w
    cell = ws2.cell(row=1, column=ci, value=h)
    cell.font = hdr()
    cell.fill = HEADER_FILL
    cell.alignment = center()
    cell.border = border()
ws2.row_dimensions[1].height = 22

row = 2
for story in STORIES:
    s_fill = DONE_FILL if story["status"] == "Done" else STORY_FILL
    row_data = [
        story["key"], story["summary"], story["type"],
        story["priority"], story["points"], story["status"],
        story["assignee"], ""
    ]
    for ci, val in enumerate(row_data, start=1):
        c = ws2.cell(row=row, column=ci, value=val)
        c.fill = s_fill
        c.font = Font(bold=True, size=10)
        c.alignment = left() if ci == 2 else center()
        c.border = border()
    row += 1

    for st in story.get("subtasks", []):
        st_fill = DONE_FILL if st["status"] == "Done" else SUBTASK_FILL
        st_data = [
            f"  ↳ {st['key']}", f"    {st['summary']}", "", "",
            "", st["status"], "", st.get("hours", "")
        ]
        for ci, val in enumerate(st_data, start=1):
            c = ws2.cell(row=row, column=ci, value=val)
            c.fill = st_fill
            c.font = Font(italic=True, size=9, color="444444")
            c.alignment = left() if ci <= 2 else center()
            c.border = border()
        row += 1

# totals row
ws2.cell(row=row, column=1, value="TOTAL").font = Font(bold=True)
ws2.cell(row=row, column=5, value=sum(s["points"] for s in STORIES)).font = Font(bold=True)
ws2.cell(row=row, column=5).fill = PatternFill("solid", fgColor="FFF2CC")

# freeze header row
ws2.freeze_panes = "A2"

# ── SHEET 3: Summary by Priority ─────────────────────────────────────────────
ws3 = wb.create_sheet("By Priority")
ws3.column_dimensions["A"].width = 14
ws3.column_dimensions["B"].width = 12
ws3.column_dimensions["C"].width = 12

for ci, h in enumerate(["Priority", "Story Count", "Total Points"], start=1):
    c = ws3.cell(row=1, column=ci, value=h)
    c.font = hdr()
    c.fill = HEADER_FILL
    c.alignment = center()
    c.border = border()

priority_order = ["Critical", "High", "Medium", "Low"]
priority_colors = {"Critical": "C00000", "High": "E26B0A", "Medium": "375623", "Low": "595959"}
for i, p in enumerate(priority_order, start=2):
    s = [x for x in STORIES if x["priority"] == p]
    ws3.cell(row=i, column=1, value=p).font = Font(bold=True, color=priority_colors[p])
    ws3.cell(row=i, column=2, value=len(s)).alignment = center()
    ws3.cell(row=i, column=3, value=sum(x["points"] for x in s)).alignment = center()
    for ci in range(1, 4):
        ws3.cell(row=i, column=ci).border = border()

# ── SAVE ─────────────────────────────────────────────────────────────────────
out_path = f".claude/SDLCs/reports/sprint-report-{SPRINT['version']}.xlsx"
wb.save(out_path)
print(f"✓ Report saved: {out_path}")
```

**6c. Execute the script:**
```bash
# Ensure openpyxl is available
pip install openpyxl --quiet 2>/dev/null || pip3 install openpyxl --quiet

# Populate script with real data from Jira + local files, then run:
python3 .claude/SDLCs/reports/_gen_sprint_report.py
```

**6d. Confirm output:**
```
✓ XLSX Report generated
  Path:    .claude/SDLCs/reports/sprint-report-{version}.xlsx
  Sheets:  Sprint Overview | Stories & Tasks | By Priority
  Stories: {N} stories | {P} total points
```

## Output
1. **Local file:** `.claude/SDLCs/releases/release-{version}.md`
2. **Local file:** `.claude/SDLCs/reports/sprint-report-{version}.xlsx` — Sprint XLSX report
3. **Jira action:** Jira Version created; release notes comment on Epic
4. **GitHub action:** Version tag `v{version}` created on `main`
5. **Summary:** Version tagged, stories included, release doc path, XLSX path

## Validation
- [ ] Sprint branch merged to `main` before release is tagged
- [ ] Release document written to `.claude/SDLCs/releases/release-{version}.md`
- [ ] Jira Version created with correct name and release date
- [ ] Release notes comment added to Epic
- [ ] All included stories verified as Done status
- [ ] GitHub tag `v{version}` created on `main`
- [ ] XLSX report generated at `.claude/SDLCs/reports/sprint-report-{version}.xlsx`
- [ ] XLSX contains 3 sheets: Sprint Overview, Stories & Tasks, By Priority

## Next Step
`/sdlc-deploy {version} staging`
