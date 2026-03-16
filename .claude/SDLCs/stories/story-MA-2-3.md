# Story: Complete a Task with Timestamp Tracking

## Details
- Story Key: MA-5
- Epic Key: MA-2
- Priority: Critical
- Story Points: 2
- Status: To Do

## User Story
As a user,
I want to mark a financial task as complete and have the completion time recorded,
So that I can track my on-time payment history and feel a sense of progress.

## Acceptance Criteria
- Given I have an active task, When I mark it complete, Then it moves out of the active list and a completion timestamp is recorded in storage
- Given I mark a task complete, When I view the completed tasks section, Then I see the task with its completion date displayed
- Given a task is completed, When the system records the event, Then the completion timestamp is compared to the due date and flagged as on-time or late
- Given I accidentally complete a task, When I undo within 5 seconds via a toast action, Then the task returns to active status and the timestamp is discarded

## Technical Notes
- Completion timestamp must be stored in UTC; display in user's local timezone
- On-time/late flag is a derived field: `completed_at <= due_date`
- Undo toast is a UX nicety — implement as a brief delay before persisting, not a true undo log

## Dependencies
- Depends on: story-MA-2-1 (task creation)
- Blocks: story-MA-2-4 (recurring — completion triggers next occurrence generation)
