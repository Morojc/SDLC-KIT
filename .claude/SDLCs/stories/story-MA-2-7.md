# Story: Onboarding Flow with Pre-Built Task Templates

## Details
- Story Key: MA-9
- Epic Key: MA-2
- Priority: Medium
- Story Points: 5
- Status: To Do

## User Story
As a new user,
I want an onboarding flow that offers pre-built financial task templates I can add in one click,
So that I can get started immediately without manually entering common recurring tasks from scratch.

## Acceptance Criteria
- Given I am a new user completing onboarding, When I reach the template selection step, Then I see at least 5 pre-built templates: Rent, Electricity Bill, Internet Bill, Monthly Savings Transfer, Credit Card Payment
- Given I select a template, When I confirm, Then a pre-populated task is added to my list without requiring additional form entry
- Given I complete onboarding, When I reach the main app, Then my selected templates appear as active tasks in the dashboard
- Given I want to skip onboarding, When I choose to skip, Then I land on the empty dashboard and can create tasks manually
- Given onboarding is complete, When I re-open the app in a future session, Then I am not shown the onboarding flow again

## Technical Notes
- Templates are hardcoded in MVP — no user-created templates yet
- Template due dates should default to end of the current month; users can edit after creation
- Track onboarding completion state in local/user storage (a simple boolean flag)

## Dependencies
- Depends on: story-MA-2-1 (task creation), story-MA-2-8 (data persistence), story-MA-2-10 (auth)
- Blocks: nothing (leaf story)
