# Story: Filter and Sort Task List

## Details
- Story Key: MA-8
- Epic Key: MA-2
- Priority: Medium
- Story Points: 3
- Status: To Do

## User Story
As a user,
I want to filter my task list by category and sort by due date or priority,
So that I can focus on a specific area of my finances without distraction.

## Acceptance Criteria
- Given I am on the task list, When I filter by a category (e.g., Bills), Then only tasks in that category are shown and the visible count updates
- Given I apply a category filter, When I select a sort order (due date or priority), Then both filter and sort are applied simultaneously
- Given I am on the task list, When I sort by due date, Then tasks are ordered chronologically with the soonest due date first
- Given I am on the task list, When I sort by priority, Then tasks are ordered High → Medium → Low
- Given I apply filter and sort settings, When I navigate away and return in the same session, Then my filter and sort preferences are preserved

## Technical Notes
- Filter and sort state should be held in URL params or session storage — not persisted to the database
- Multi-category filter (select more than one category) is out of scope for MVP

## Dependencies
- Depends on: story-MA-2-1 (task creation), story-MA-2-5 (task list view)
- Blocks: nothing (leaf story)
