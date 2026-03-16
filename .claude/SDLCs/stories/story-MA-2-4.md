# Story: Recurring Tasks with Auto-Generated Occurrences

## Details
- Story Key: MA-6
- Epic Key: MA-2
- Priority: High
- Story Points: 8
- Status: To Do

## User Story
As a user,
I want to set a task to recur on a schedule and have the next occurrence auto-created when I complete one,
So that I never have to manually recreate monthly bills or savings transfers.

## Acceptance Criteria
- Given I create a task with recurrence set to weekly, monthly, or custom interval, When I mark it complete, Then a new task is automatically created with the next due date calculated correctly
- Given a recurring task is completed, When the next occurrence is generated, Then all fields (title, amount, category, priority) are copied and only the due date is incremented
- Given a monthly recurring task is due on the 31st, When the next month has fewer than 31 days, Then the next due date falls on the last day of that month
- Given I want to stop a recurring series, When I delete the task, Then I am asked whether to delete just this occurrence or all future occurrences
- Given I edit a recurring task's fields, When I save, Then I am asked whether to apply changes to this occurrence only or all future occurrences
- Given a custom recurrence (e.g., every 2 weeks), When I complete the task, Then the next due date is exactly 14 days after the current due date (not after the completion date)

## Technical Notes
- Recurrence engine should calculate next due date from the original due date, not from completion date — prevents drift
- Supported frequencies for MVP: Weekly, Bi-weekly, Monthly, Quarterly, Custom (N days)
- End-of-month normalization: use `date-fns` or equivalent; test Feb 28/29 edge cases explicitly
- Store recurrence rule as a structured object, not a string, to allow future iCal export

## Dependencies
- Depends on: story-MA-2-1 (task creation), story-MA-2-3 (completion trigger)
- Blocks: nothing (leaf story)
