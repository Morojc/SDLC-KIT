# sdlc-qa — SDLC Phase 5: Testing

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/plans/plan-{key}.md` — implementation plan (Definition of Done)
- `.claude/SDLCs/test-reports/test-{key}.md` — test report (if exists)
- `.claude/SDLCs/designs/design-{key}.md` — design document (acceptance criteria)
- `CLAUDE.md` (if exists) — project QA conventions

## Input
$ARGUMENTS — A Jira story key (e.g. `PROJ-010`) or release key (e.g. `PROJ-VERSION-1`)

## Task
Run a comprehensive QA checklist against the acceptance criteria, validate code quality gates, and transition the Jira ticket to "In Review" upon passing.

### Step 1: Load Acceptance Criteria
Fetch the story or release from Jira:
```
GET $JIRA_BASE_URL/rest/api/3/issue/{key}
Authorization: Basic base64($JIRA_EMAIL:$JIRA_API_TOKEN)
```
Extract:
- Acceptance criteria from `customfield_10016`
- Story description (additional requirements)
- Linked issues (sub-tasks, bugs)
- Story points and assignee

If the argument is a release key, also fetch all stories in the release via JQL:
```
GET $JIRA_BASE_URL/rest/api/3/search?jql=fixVersion="{version}" AND status="In Review"
```

### Step 2: Run Automated Quality Gates
Execute all quality gates and record results:

**2a. Tests:**
```bash
bun test --coverage
```
Record: total tests, passed, failed, coverage percentage per file.

**2b. Lint:**
```bash
bun run lint
```
Record: number of errors, number of warnings.

**2c. Type check:**
```bash
bun run typecheck
```
Record: number of type errors.

**2d. Build:**
```bash
bun run build
```
Record: success or failure, any warnings.

**GATE:** All four checks must pass (zero failures, zero errors) before proceeding to manual checks.

### Step 3: Verify Acceptance Criteria
For each acceptance criterion in the story, perform a manual verification:

Format for each criterion:
```
AC-{N}: {criterion text}
Test: {how you verified it — specific command, screen, or logic review}
Result: PASS / FAIL / PARTIAL
Notes: {any caveats or edge cases observed}
```

Verification methods:
- For behavior specs: trace the code path from entry point to output
- For UI specs: describe the expected DOM state or user interaction
- For API specs: construct a sample request and trace the response
- For performance specs: check if thresholds are enforced in tests or config

### Step 4: Check Definition of Done
Review the plan's Definition of Done checklist:
- [ ] All tests pass with coverage ≥ 80%
- [ ] No lint errors
- [ ] No TypeScript errors
- [ ] PR exists with story key in title/description
- [ ] All sub-tasks are Done in Jira
- [ ] No open blocking bugs linked to this story
- [ ] Code reviewed (or self-reviewed against conventions)
- [ ] Documentation updated (if public API/behavior changed)

### Step 5: Write QA Report
Write the QA results to `.claude/SDLCs/test-reports/qa-{key}.md`:

```markdown
# QA Report: {key} — {Story Title}

## Date: {YYYY-MM-DD}
## QA By: Claude (automated) + human sign-off required

## Quality Gate Results
| Gate | Status | Details |
|------|--------|---------|
| Tests | ✅ PASS / ❌ FAIL | {N} passed, {N} failed, {pct}% coverage |
| Lint | ✅ PASS / ❌ FAIL | {N} errors, {N} warnings |
| Typecheck | ✅ PASS / ❌ FAIL | {N} type errors |
| Build | ✅ PASS / ❌ FAIL | {details} |

## Acceptance Criteria Verification
| AC | Criterion | Status | Notes |
|----|-----------|--------|-------|
| AC-1 | {text} | ✅ PASS | {notes} |
| AC-2 | {text} | ✅ PASS | {notes} |

## Definition of Done
- [x] All tests pass
- [x] Coverage ≥ 80%
- [x] No lint errors
- [x] No TypeScript errors
- [x] Sub-tasks Done
- [ ] PR approved (requires human)

## Issues Found
{List any bugs or gaps found during QA, with severity}
| Issue | Severity | Action |
|-------|----------|--------|
| {description} | {High/Med/Low} | File bug / Fix inline / Accept risk |

## QA Verdict
**{PASS / FAIL / CONDITIONAL PASS}**
{1-2 sentence rationale}

## Outstanding Risks
{Any known limitations or deferred items}
```

### Step 6: Handle Failures
If any acceptance criterion fails or a quality gate fails:
- Do NOT transition to In Review
- For each failure: describe the gap and suggest the fix
- If the fix is within the current story scope, implement it now and re-run the gate
- If the fix requires a new ticket, use `/sdlc-bug "{description}"` to log it

### Step 7: Jira Sync (on PASS)
**7a. Transition the story to "In Review":**
First, get available transitions:
```
GET $JIRA_BASE_URL/rest/api/3/issue/{key}/transitions
```
Find the transition ID for "In Review", then:
```
POST $JIRA_BASE_URL/rest/api/3/issue/{key}/transitions
{ "transition": { "id": "{in_review_transition_id}" } }
```

**7b. Add QA summary comment:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{key}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "QA complete. All {N} acceptance criteria verified. Tests: {pct}% coverage. Lint: clean. Status: PASS. Ready for human review." }
    ]}]
  }
}
```

**7c. Update the QA sub-task to Done (if it exists):**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{qaSubTaskKey}/transitions
{ "transition": { "id": "{done_transition_id}" } }
```

## Output
1. **QA report:** `.claude/SDLCs/test-reports/qa-{key}.md`
2. **Jira actions (on PASS):**
   - Story transitioned to "In Review"
   - QA summary comment added
   - QA sub-task marked Done
3. **Summary:** PASS/FAIL verdict with gate results table

## Validation
- [ ] QA report written to `.claude/SDLCs/test-reports/qa-{key}.md`
- [ ] All quality gates documented with actual results (not assumed)
- [ ] Every acceptance criterion has a verified status
- [ ] Story is "In Review" in Jira (only if QA verdict is PASS)
- [ ] Any failures logged as bugs or fixed inline

## Next Step
After completion (PASS), suggest: `/sdlc-release {version} {epicKey}`
After completion (FAIL), suggest: `/sdlc-implement {planFile}` to fix the issues, then re-run `/sdlc-qa {key}`
