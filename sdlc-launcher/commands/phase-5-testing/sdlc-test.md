# sdlc-test — SDLC Phase 5: Testing

## Context Loading
Read and internalize:
- `.claude/settings/jira-config.json` — Jira project settings
- `.claude/SDLCs/plans/plan-{storyKey}.md` — implementation plan (file targets, validation commands)
- `.claude/SDLCs/designs/design-{storyKey}.md` — design document (data contracts, edge cases)
- `CLAUDE.md` (if exists) — testing conventions, coverage thresholds, test framework

## Input
$ARGUMENTS — A Jira story key (e.g. `PROJ-010`) or a specific file path to test (e.g. `src/api/sync.ts`)

## Task
Generate a comprehensive test plan, write unit, integration, and e2e tests for the story or file, and create a QA sub-task in Jira.

### Step 1: Identify What to Test
If the argument is a Jira key:
- Read the plan at `.claude/SDLCs/plans/plan-{storyKey}.md` — extract File Targets
- Fetch the story's acceptance criteria from Jira:
  ```
  GET $JIRA_BASE_URL/rest/api/3/issue/{storyKey}
  ```
  Extract `customfield_10016` (acceptance criteria) and `description`

If the argument is a file path:
- Derive the story key from the plan files or git log
- Read the target file directly

### Step 2: Analyze Existing Tests
- Find all existing test files with patterns: `**/*.spec.ts`, `**/*.test.ts`, `**/__tests__/**`
- Read tests for files adjacent to the implementation to understand testing conventions:
  - Test file naming and location (co-located vs `__tests__` dir)
  - Assertion library (expect, assert, etc.)
  - Mocking approach (jest.mock, sinon, etc.)
  - Fixture/factory patterns used

### Step 3: Build the Test Plan
Design three layers of tests:

**Unit Tests** — test individual functions/classes in isolation
- One describe block per public function or class method
- Positive path, edge cases, error cases
- Mock all external dependencies (DB, network, filesystem)

**Integration Tests** — test how components work together
- API endpoint → service → repository chain
- Database interactions (use test DB or in-memory)
- Message queue/event flows

**End-to-End Tests** (if applicable)
- User-facing flows matching acceptance criteria scenarios
- Browser automation (Playwright/Cypress) or CLI scenarios

### Step 4: Write the Tests
For each test file:
- Place test files according to project conventions (co-located or `__tests__`)
- Use `describe` / `it` / `test` blocks with descriptive names
- Every acceptance criterion scenario should map to at least one test case
- Include a test for each error/edge case in the implementation

Example structure:
```typescript
describe('{ModuleName}', () => {
  describe('{methodName}', () => {
    it('should {positive case description}', async () => { ... });
    it('should {edge case description}', async () => { ... });
    it('should throw {ErrorType} when {condition}', async () => { ... });
  });
});
```

### Step 5: Run Tests and Verify Coverage
```bash
bun test --coverage      # Run all tests with coverage
bun run lint             # Lint test files too
```

Coverage target: **≥ 80%** on all new files. If below threshold:
- Add missing test cases
- Re-run until coverage target is met

### Step 6: Write Test Report
Write a test summary to `.claude/SDLCs/test-reports/test-{storyKey}.md`:

```markdown
# Test Report: {storyKey} — {Story Title}

## Coverage Summary
| File | Statements | Branches | Functions | Lines |
|------|------------|----------|-----------|-------|
| {file} | {pct}% | {pct}% | {pct}% | {pct}% |

## Test Results
- Total tests: {N}
- Passed: {N}
- Failed: 0
- Skipped: {N}

## Test Files Written
- `{test file path}` — {N} tests ({unit/integration/e2e})

## Acceptance Criteria Coverage
| Criterion | Test | Status |
|-----------|------|--------|
| {AC text} | `it('{test name}')` | ✅ |

## Known Gaps
{Any scenarios not covered and why}
```

### Step 7: Jira Sync
**7a. Create a QA sub-task under the story:**
```
POST $JIRA_BASE_URL/rest/api/3/issue
{
  "fields": {
    "project": { "key": "{PROJECT_KEY}" },
    "summary": "QA: Test coverage for {storyKey}",
    "issuetype": { "name": "Sub-task" },
    "parent": { "key": "{storyKey}" },
    "description": {
      "type": "doc", "version": 1,
      "content": [{ "type": "paragraph", "content": [
        { "type": "text", "text": "{N} tests written. Coverage: {pct}%. Report: .claude/SDLCs/test-reports/test-{storyKey}.md" }
      ]}]
    },
    "labels": ["sdlc-plugin", "phase-5-testing", "qa"]
  }
}
```

**7b. Add test summary as comment on the story:**
```
POST $JIRA_BASE_URL/rest/api/3/issue/{storyKey}/comment
{
  "body": {
    "type": "doc", "version": 1,
    "content": [{ "type": "paragraph", "content": [
      { "type": "text", "text": "Tests written: {N} ({unit} unit, {integration} integration). Coverage: {pct}%. All tests passing." }
    ]}]
  }
}
```

## Output
1. **Test files:** Written to appropriate locations per project conventions
2. **Test report:** `.claude/SDLCs/test-reports/test-{storyKey}.md`
3. **Jira actions:**
   - QA sub-task created under the story
   - Test summary comment added to story
4. **Summary:** Test count, coverage percentage, and any gaps

## Validation
- [ ] All tests pass (`bun test`)
- [ ] Coverage ≥ 80% on new files
- [ ] Every acceptance criterion has at least one corresponding test
- [ ] Test report written to `.claude/SDLCs/test-reports/test-{storyKey}.md`
- [ ] QA sub-task created in Jira
- [ ] Test summary comment on story

## Next Step
After completion, suggest: `/sdlc-qa {storyKey}`
