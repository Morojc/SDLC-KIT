# Story: Create a Financial Task

## Details
- Story Key: MA-3
- Epic Key: MA-2
- Priority: Critical
- Story Points: 3
- Status: To Do

## User Story
As a user,
I want to create a financial task with a title, amount, due date, category, and priority,
So that I have a clear record of what financial action I need to take and when.

## Acceptance Criteria
- Given I am on the task creation screen, When I fill in all required fields (title, amount, due date, category, priority) and submit, Then the task appears in my task list with all entered data saved correctly
- Given I am creating a task, When I leave a required field empty and submit, Then I see an inline validation error on that field and the task is not saved
- Given I am creating a task, When I select a category, Then I can choose from: Bills, Savings, Debt, Income, Other
- Given I am creating a task, When I set a due date in the past, Then I see a warning (not an error) indicating the task will be immediately overdue

## Technical Notes
- Amount field should accept decimal input (up to 2 decimal places) and display in user's locale currency format
- Category should be a controlled enum — no free-text category creation in MVP
- Priority levels: High, Medium, Low

## Dependencies
- Depends on: Auth/identity layer (story-MA-2-10) and data persistence layer (story-MA-2-8)
- Blocks: story-MA-2-2 (edit/delete), story-MA-2-3 (completion), story-MA-2-4 (recurring), story-MA-2-5 (dashboard)
