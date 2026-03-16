# Story: Install App as PWA on Mobile

## Details
- Story Key: MA-11
- Epic Key: MA-2
- Priority: Medium
- Story Points: 3
- Status: To Do

## User Story
As a user visiting the app on my mobile browser,
I want to install it to my home screen as a PWA,
So that I can access my financial tasks quickly without opening a browser.

## Acceptance Criteria
- Given I visit the app on a supported mobile browser (Chrome Android, Safari iOS), When the PWA install criteria are met, Then the browser shows an install prompt or I can install via the browser menu
- Given I have installed the PWA, When I open it from the home screen, Then it launches in standalone mode with no browser chrome
- Given I am offline and open the installed PWA, When the app loads, Then the app shell and my cached tasks are accessible and readable
- Given I am offline, When I attempt to create or complete a task, Then the action is queued (per story-MA-2-8) and the UI confirms the offline state clearly

## Technical Notes
- Requires: `manifest.json` with correct display, icons (192×192, 512×512), theme color
- Requires: Service worker with cache-first strategy for app shell; network-first for task data
- Test on: Chrome Android 110+, Safari iOS 16+ (Safari has limited PWA support — document known gaps)
- Lighthouse PWA audit score target: ≥90

## Dependencies
- Depends on: story-MA-2-8 (data persistence / offline queue)
- Blocks: nothing (leaf story)
