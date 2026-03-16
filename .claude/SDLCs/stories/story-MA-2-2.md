# Story: Edit and Delete Financial Tasks

## Details
- Story Key: MA-4
- Epic Key: MA-2
- Priority: High
- Story Points: 2
- Status: To Do

## User Story
As a user,
I want to edit any field on an existing task or delete it entirely,
So that I can keep my task list accurate as my financial situation changes.

## Acceptance Criteria
- Given I have an existing task, When I edit any field and save, Then the updated values are reflected immediately in the task list and dashboard
- Given I have an existing task, When I choose to delete it, Then I am shown a confirmation prompt before deletion
- Given I confirm deletion, When the action completes, Then the task is permanently removed from all views
- Given I cancel the deletion prompt, When I return to the list, Then the task is unchanged
- Given I edit a recurring task, When I save changes, Then I am asked whether to update this occurrence only or all future occurrences

## Technical Notes
- Edit should reuse the same form component as task creation
- Deletion is hard-delete in MVP (no soft-delete/archive); reconsider in v1.1

## Dependencies
- Depends on: story-MA-2-1 (task creation)
- Blocks: nothing (leaf story)
