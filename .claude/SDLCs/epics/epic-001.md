# Epic: Build Personal Finance Task Tracker MVP

## Jira Reference
- Epic Key: MA-2
- Initiative Key: MA-1
- Created: 2026-03-16
- Status: To Do
- Estimated Size: L

## Business Value
Millions of individuals lose money or accumulate stress from missed bills and forgotten savings goals because existing tools (spreadsheets, banking apps) are passive — they report on finances rather than driving action. A task-oriented finance app converts financial intentions into completed actions, directly reducing missed payments and improving savings follow-through. For the target user, this means fewer late fees, less mental overhead, and measurable progress on financial goals.

## Scope

### In Scope
- Task creation for financial actions (pay bill, transfer savings, review subscription, etc.)
- Task fields: title, amount, due date, category (bills, savings, debt, income), priority, completion status
- Recurring task support with configurable frequency (weekly, monthly, custom)
- Dashboard view: overdue, due today, upcoming tasks
- Category-based filtering and sorting
- Basic onboarding with pre-built task templates (rent, utilities, subscriptions)
- Single-user, local/account-based data storage (no bank integration)

### Out of Scope
- Bank account or credit card syncing/integration
- Multi-user or shared household finance features
- Investment tracking or portfolio management
- Budgeting charts, spending analytics, or reporting dashboards
- Push/SMS/email notifications (deferred to future phase)
- Mobile native app (web-first MVP)

## Acceptance Criteria
- [ ] A user can create, edit, complete, and delete financial tasks with all required fields (title, amount, due date, category, priority)
- [ ] Recurring tasks auto-generate the next occurrence upon completion
- [ ] The dashboard surfaces overdue and today's tasks prominently without additional navigation
- [ ] Users can filter tasks by category and sort by due date or priority
- [ ] Onboarding flow offers at least 5 pre-built task templates that can be added in one click
- [ ] ≥80% of scheduled tasks are completed on time by active users within 30 days (tracked via completion timestamps)

## Dependencies
- Auth/user identity system (or local storage strategy decision)
- UI component library selection (affects development speed)
- Data persistence layer (local-first vs. cloud backend decision required before development)

## Stories Overview
[To be populated by /sdlc-stories]

## Notes
- Complexity rated **L**: Core CRUD + recurring logic + dashboard UX is non-trivial but well-scoped; no external integrations in MVP
- Bank sync is explicitly deferred — clear scope communication in onboarding is critical to prevent user disappointment
- Consider progressive web app (PWA) approach to bridge web-first and mobile needs without a separate native build
