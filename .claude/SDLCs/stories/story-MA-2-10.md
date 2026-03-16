# Story: User Authentication and Identity

## Details
- Story Key: MA-12
- Epic Key: MA-2
- Priority: Critical
- Story Points: 5
- Status: To Do

## User Story
As a user,
I want to create an account and log in securely,
So that my financial tasks are private and accessible only to me.

## Acceptance Criteria
- Given I am a new user, When I sign up with an email and password, Then my account is created and I am logged in automatically
- Given I am a returning user, When I enter valid credentials, Then I am authenticated and redirected to my dashboard
- Given I enter invalid credentials, When I attempt to log in, Then I see a generic error ("Invalid email or password") without revealing which field is wrong
- Given I am authenticated, When I close and reopen the app, Then I remain logged in (persistent session via secure cookie or token)
- Given my session expires, When I attempt any authenticated action, Then I am redirected to login without data loss
- Given I log out, When the action completes, Then my session is invalidated server-side and local tokens are cleared

## Technical Notes
- Auth strategy decision pending (open question, due 2026-04-30): third-party (Clerk, Auth0) preferred over hand-rolled
- Tokens must be stored in httpOnly cookies — never in localStorage (per NFR security requirement)
- Password reset flow is in scope; OAuth/social login is out of scope for MVP
- If local-first architecture is chosen (no cloud), auth can be simplified to a local PIN or skipped entirely — revisit after architecture decision

## Dependencies
- Depends on: architecture decision (open question, due 2026-04-30)
- Blocks: story-MA-2-1, story-MA-2-7, story-MA-2-8 (all stories requiring user-scoped data)
