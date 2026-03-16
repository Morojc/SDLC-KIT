# Product Requirements Document
## Epic: MA-2 — Build Personal Finance Task Tracker MVP

## Overview
A personal finance task tracker that converts financial intentions into actionable to-dos with due dates, amounts, and categories — giving users a clear, task-oriented view of what financial actions need to be taken and when. The MVP targets individuals aged 20–45 who want to stop missing bills and savings goals without the complexity of a full budgeting suite. Success is measured by on-time task completion rates and daily active usage after onboarding.

## Problem & Opportunity
Managing personal finances is fragmented — users juggle spreadsheets, banking apps, and mental notes with no single tool that treats financial actions as completable to-dos. The result: missed bill payments, forgotten savings transfers, and chronic financial stress. There is a gap between complex budgeting tools (Mint, YNAB) and bare-bones reminder apps — a task-first, finance-focused tool fills that gap with minimal setup friction.

**User impact:**
- Missed bill payments cost users late fees (avg $25–$40/incident)
- 60% of adults report financial stress as a top life stressor (APA, 2023)
- Target: reduce missed bill payments to 0 and bring on-time completion rate to ≥80% within 30 days

## Goals & Non-Goals

### Goals
- Users complete ≥80% of scheduled financial tasks on time within 30 days of adoption
- Zero missed bill payments for active users within 60 days
- Daily active usage ≥3 days/week after 2-week onboarding
- Full task lifecycle (create → track → complete) works offline via PWA

### Non-Goals
- Bank account / credit card syncing or read access to financial accounts
- Multi-user, shared household, or collaborative finance features
- Investment tracking, portfolio management, or net worth calculation
- Budgeting charts, spending trend analytics, or reporting dashboards
- Push/SMS/email notifications (deferred to v1.1)
- Native iOS / Android app (web-first PWA for MVP)

## User Stories Summary
| Story Key | Title | Priority | Est. Points |
|-----------|-------|----------|-------------|
| TBD | As a user, I want to create a financial task with title, amount, due date, and category | High | 3 |
| TBD | As a user, I want to edit and delete existing tasks | High | 2 |
| TBD | As a user, I want to mark a task complete and see it removed from my active list | High | 2 |
| TBD | As a user, I want recurring tasks to auto-generate the next occurrence on completion | High | 5 |
| TBD | As a user, I want a dashboard that shows overdue, due today, and upcoming tasks | High | 5 |
| TBD | As a user, I want to filter tasks by category and sort by due date or priority | Medium | 3 |
| TBD | As a new user, I want an onboarding flow with pre-built task templates | Medium | 3 |
| TBD | As a user, I want to install the app on my phone via PWA | Medium | 2 |
| TBD | As a user, I want my data to persist across sessions and devices | High | 5 |

## Functional Requirements

### FR-001: Task Creation
**Description:** Users can create a financial task with all required fields.
**Acceptance Criteria:**
- Given I am on the task creation screen, When I fill in title, amount, due date, category, and priority and submit, Then a new task appears in my task list with all entered data saved
- Given I am creating a task, When I leave a required field empty and submit, Then I see an inline validation error and the task is not saved
- Given I am creating a task, When I select a category, Then I can choose from: Bills, Savings, Debt, Income, Other

### FR-002: Task Editing & Deletion
**Description:** Users can modify or remove any existing task.
**Acceptance Criteria:**
- Given I have an existing task, When I edit any field and save, Then the updated values are reflected immediately in the task list and dashboard
- Given I have an existing task, When I choose to delete it and confirm, Then the task is permanently removed from all views
- Given I choose to delete a task, When I am shown a confirmation prompt and cancel, Then the task remains unchanged

### FR-003: Task Completion
**Description:** Users can mark tasks complete; completed tasks are tracked with a timestamp.
**Acceptance Criteria:**
- Given I have an active task, When I mark it complete, Then it moves out of the active list and a completion timestamp is recorded
- Given I have marked a task complete, When I view completed tasks, Then I can see the task with its completion date
- Given a task is completed, When the system records the event, Then the timestamp is used to calculate on-time vs. late completion relative to the due date

### FR-004: Recurring Tasks
**Description:** Tasks can be set to recur at configurable intervals; the next occurrence is auto-generated on completion.
**Acceptance Criteria:**
- Given I create a task with recurrence set to weekly/monthly/custom, When I mark it complete, Then a new task is automatically created with the next due date calculated correctly
- Given a recurring task is completed, When the next occurrence is generated, Then all fields (title, amount, category, priority) are copied and only the due date is incremented
- Given a recurring monthly task is due on the 31st, When the next month has fewer days, Then the due date falls on the last day of that month
- Given I want to stop a recurring task, When I delete it, Then I am asked whether to delete just this occurrence or all future occurrences

### FR-005: Dashboard View
**Description:** The home dashboard surfaces the most time-sensitive tasks without additional navigation.
**Acceptance Criteria:**
- Given I open the app, When the dashboard loads, Then I see three sections: Overdue, Due Today, and Upcoming (next 7 days)
- Given I have overdue tasks, When I view the dashboard, Then overdue tasks are visually distinguished (e.g., red indicator) and appear first
- Given all tasks are up to date, When I view the dashboard, Then I see a positive empty state rather than a blank screen

### FR-006: Filtering & Sorting
**Description:** Users can filter and sort their task list to focus on specific areas.
**Acceptance Criteria:**
- Given I am on the task list, When I filter by a category, Then only tasks in that category are shown and the count updates
- Given I am on the task list, When I sort by due date, Then tasks are ordered chronologically (soonest first by default)
- Given I am on the task list, When I sort by priority, Then tasks are ordered High → Medium → Low
- Given I apply a filter and a sort together, Then both are applied simultaneously and persist during the session

### FR-007: Onboarding & Task Templates
**Description:** New users are guided through setup with pre-built financial task templates.
**Acceptance Criteria:**
- Given I am a new user completing onboarding, When I reach the template step, Then I see at least 5 pre-built templates: Rent, Electricity Bill, Internet Bill, Monthly Savings Transfer, Credit Card Payment
- Given I select a template during onboarding, When I confirm, Then a pre-populated task is added to my list in one action (no additional form entry required)
- Given I complete onboarding, When I reach the main app, Then my selected templates appear as active tasks ready for customization

### FR-008: Data Persistence & Sync
**Description:** User data persists across sessions; architecture supports optional future cloud sync.
**Acceptance Criteria:**
- Given I create tasks in one session, When I close and reopen the app, Then all tasks are present with no data loss
- Given the app is offline, When I complete or create a task, Then the action is queued and applied when connectivity returns (PWA offline support)
- Given I access the app on a second device (post-MVP with cloud backend), When I log in, Then my tasks are synchronized

### FR-009: PWA Installability
**Description:** The app can be installed on mobile devices as a progressive web app.
**Acceptance Criteria:**
- Given I visit the app on a mobile browser, When prompted or via browser menu, Then I can install it to my home screen
- Given the app is installed as a PWA, When I open it, Then it launches in standalone mode (no browser chrome)
- Given I am offline, When I open the installed PWA, Then the app shell and cached tasks are accessible

---

## Non-Functional Requirements
| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | App initial load (web) | < 2s on mid-range mobile (Lighthouse score ≥90) |
| Performance | Task create/update response | < 300ms perceived latency |
| Performance | Dashboard render with 100+ tasks | < 500ms |
| Security | Auth tokens | Stored in httpOnly cookies or secure storage (never localStorage) |
| Security | User data isolation | No user can access another user's data |
| Security | Input sanitization | All user inputs sanitized before storage; XSS prevention enforced |
| Scalability | Single-user data volume | Handle ≥1,000 tasks per user without performance degradation |
| Scalability | Cloud backend (future) | Architecture must not preclude migration from local-first to cloud |
| Availability | Web app uptime | 99.5% (MVP tolerance; 99.9% post-launch) |
| Accessibility | WCAG compliance | Level AA — keyboard navigation, screen reader support, color contrast |
| Accessibility | Font sizing | Minimum 16px body text; user browser zoom must not break layout |
| Offline | PWA offline mode | Core read + write task operations available offline |

---

## Technical Constraints
- **Web-first:** Must run in modern browsers (Chrome 110+, Firefox 115+, Safari 16+); no IE support
- **PWA-first mobile:** No separate native app build for MVP; service worker required for offline and installability
- **Architecture decision pending:** Local-first (IndexedDB / SQLite via WASM) vs. cloud backend (REST API + DB) — must be decided by end of April 2026 before any storage code is written
- **UI component library:** To be selected in Q2 2026; choice affects component API and accessibility baseline
- **Single-user MVP:** Auth system must be lightweight; third-party auth (Clerk, Auth0) preferred over hand-rolled to reduce surface area
- **No external integrations:** No bank APIs, no payment processors, no third-party data sources in MVP

---

## Out of Scope
- Bank account / credit card syncing (Plaid, Teller, or similar)
- Multi-user / household sharing
- Investment and portfolio tracking
- Spending analytics, charts, or trend reporting
- Push, SMS, or email notifications
- Native iOS / Android apps
- Admin dashboard or ops tooling

---

## Timeline
| Phase | Quarter | Milestone |
|-------|---------|-----------|
| Foundation | Q2 2026 (Apr–Jun) | Auth + task CRUD + persistence end-to-end |
| Core Features | Q3 2026 (Jul–Sep) | All 9 FR implemented; all AC met in staging |
| Polish & Beta | Q4 2026 (Oct–Dec) | QA passed; ≥80% on-time rate in 20–50 user beta |
| Public Launch | Q1 2027 (Jan–Mar) | v1.0 live; analytics instrumented; v1.1 backlog ready |

---

## Open Questions
- [ ] **Architecture decision — local-first vs. cloud backend** | Owner: Tech Lead | Due: 2026-04-30 | Blocks: FR-008, all storage work
- [ ] **UI component library selection** | Owner: Frontend Lead | Due: 2026-04-30 | Blocks: all UI development
- [ ] **Auth strategy — third-party (Clerk/Auth0) vs. custom** | Owner: Tech Lead | Due: 2026-04-30 | Blocks: FR-008 user isolation
- [ ] **PWA offline conflict resolution** — if user edits offline and online simultaneously post-cloud migration, what wins? | Owner: Product | Due: 2026-06-30
- [ ] **Recurrence edge case policy** — when a monthly task is due on 31st and month is shorter, use last day or skip? | Owner: Product | Due: 2026-06-01
- [ ] **Template customization scope** — can users edit template fields during onboarding, or only after? | Owner: Product | Due: 2026-05-15

---

## Jira Reference
- Epic Key: MA-2
- Initiative Key: MA-1
- PRD Created: 2026-03-16
- Status: Draft
