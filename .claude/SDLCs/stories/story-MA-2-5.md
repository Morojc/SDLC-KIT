# Story: Dashboard with Overdue, Due Today, and Upcoming Sections

## Details
- Story Key: MA-7
- Epic Key: MA-2
- Priority: Critical
- Story Points: 5
- Status: To Do

## User Story
As a user,
I want a home dashboard that groups my tasks by urgency (overdue, due today, upcoming),
So that I can immediately see what financial actions require my attention without scanning a flat list.

## Acceptance Criteria
- Given I open the app, When the dashboard loads, Then I see three distinct sections: Overdue, Due Today, and Upcoming (next 7 days)
- Given I have overdue tasks, When I view the dashboard, Then overdue tasks are visually distinguished (e.g., red indicator) and the section appears first
- Given I have no overdue or due-today tasks, When I view the dashboard, Then I see a positive empty state (e.g., "You're all caught up!") rather than blank sections
- Given a task's due date is today, When I view the dashboard at any time of day in my local timezone, Then it appears in "Due Today" — not "Overdue"
- Given I complete a task from the dashboard, When the action is confirmed, Then the task disappears from the dashboard in real time without a page reload

## Technical Notes
- Dashboard must use local timezone for date bucketing — do not compare in UTC
- "Upcoming" is a rolling 7-day window beyond today; tasks beyond 7 days are visible in full task list only
- Dashboard should render in <500ms with up to 100 tasks; consider virtual list if needed

## Dependencies
- Depends on: story-MA-2-1 (task creation), story-MA-2-3 (task completion)
- Blocks: story-MA-2-6 (filters/sorting builds on task list view)
