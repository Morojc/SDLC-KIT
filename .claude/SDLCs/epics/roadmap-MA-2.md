# Roadmap: Build Personal Finance Task Tracker MVP

## Epic Reference
- Epic Key: MA-2
- Initiative Key: MA-1
- Planning Horizon: Q2 2026 – Q1 2027
- Created: 2026-03-16

---

## Quarterly Plan

### Q2 2026 (Apr – Jun) — Foundation
**Goal:** Core infrastructure is in place and a user can create, manage, and complete financial tasks end-to-end.

**Deliverables:**
- [ ] Architecture decision: local-first (IndexedDB/SQLite) vs. cloud backend — documented and approved
- [ ] UI component library selected and design system bootstrapped
- [ ] User auth / identity layer implemented (or local profile strategy confirmed)
- [ ] Task CRUD: create, edit, complete, delete with all fields (title, amount, due date, category, priority)
- [ ] Data persistence layer wired up and tested
- [ ] Basic task list view (no dashboard polish yet)

**Exit Criteria:** A user can sign up/log in, create a financial task with all required fields, mark it complete, and have data persist across sessions.

**Dependencies:**
- Architecture decision (local-first vs. cloud) must be made by end of April — blocks all storage work
- UI component library selection must be finalized before UI development begins

**Risks:**
- Risk: Local-first vs. cloud debate stalls development | Mitigation: timebox decision to 1 week; default to local-first if unresolved
- Risk: Auth complexity underestimated for single-user scope | Mitigation: evaluate third-party auth (Clerk, Auth0) vs. simple email/password

---

### Q3 2026 (Jul – Sep) — Core Features
**Goal:** All MVP features are implemented and the app is functionally complete for a single user.

**Deliverables:**
- [ ] Recurring task engine: auto-generate next occurrence on completion, configurable frequency (weekly, monthly, custom)
- [ ] Dashboard view: overdue, due today, and upcoming task sections
- [ ] Category-based filtering and sort by due date / priority
- [ ] Onboarding flow with ≥5 pre-built task templates (rent, utilities, subscriptions, savings transfer, debt payment)
- [ ] Task completion timestamp tracking (required for success metric: ≥80% on-time completion rate)
- [ ] PWA manifest + service worker for offline capability and mobile installability

**Exit Criteria:** All 6 acceptance criteria from epic-001.md are met in a staging environment; recurring tasks, dashboard, filters, and onboarding all functional.

**Dependencies:**
- Q2 Foundation complete (auth + persistence layer must be stable)
- PWA approach confirmed (noted in epic as preferred over native mobile)

**Risks:**
- Risk: Recurring task edge cases (leap years, skipped months, timezone drift) cause bugs | Mitigation: dedicate explicit QA sprint to recurrence logic
- Risk: Dashboard UX requires multiple iteration rounds, delaying feature completion | Mitigation: prototype dashboard in Figma and validate with 3 test users before build

---

### Q4 2026 (Oct – Dec) — Polish & Beta
**Goal:** App is stable, performant, and ready for real users; beta cohort validates success metrics.

**Deliverables:**
- [ ] End-to-end QA pass: all acceptance criteria verified, edge cases documented
- [ ] Performance audit: app loads in <2s on mid-range mobile device
- [ ] Accessibility audit: WCAG 2.1 AA compliance for core flows
- [ ] Beta launch to 20–50 invited users (target: individuals aged 20–45)
- [ ] In-app feedback collection mechanism (simple form or rating prompt)
- [ ] Completion rate tracking dashboard for internal metrics (not user-facing)
- [ ] Bug bash and fix cycle based on beta feedback
- [ ] Scope communication in onboarding: clear messaging that bank sync is not available

**Exit Criteria:** Beta cohort shows ≥80% of scheduled tasks completed on time within 30 days; <5 critical bugs open; performance and accessibility targets met.

**Dependencies:**
- Q3 feature complete (all AC met before QA pass begins)
- Recruiting beta users (can begin outreach in Q3)

**Risks:**
- Risk: Beta users expect bank sync and disengage | Mitigation: onboarding explicitly sets expectations; collect feedback to inform post-MVP roadmap
- Risk: Performance issues on PWA harder to resolve than anticipated | Mitigation: run Lighthouse audits from Q3 to catch early

---

### Q1 2027 (Jan – Mar) — Public Launch & Iteration
**Goal:** Public launch with stable v1.0; first post-launch iteration based on real user data.

**Deliverables:**
- [ ] v1.0 public release (web app, PWA installable)
- [ ] Launch marketing: product hunt, personal finance communities, landing page
- [ ] Analytics instrumentation (task creation rate, completion rate, DAU/WAU)
- [ ] Post-launch bug fix cycle (2-week sprint)
- [ ] Backlog prioritization for v1.1 based on beta + launch feedback
- [ ] Evaluate: push notification support (deferred from MVP) — decision on whether to build in v1.1
- [ ] Evaluate: bank sync feasibility — research Plaid/Teller integration cost and complexity

**Exit Criteria:** App publicly live; ≥3 days/week DAU for cohort users after 2-week onboarding; v1.1 backlog prioritized with stakeholder sign-off.

**Dependencies:**
- Q4 beta success (metrics must hit thresholds before committing to public launch date)
- Analytics setup must be in place before launch to capture day-0 data

**Risks:**
- Risk: Low organic discoverability post-launch | Mitigation: plan launch channels in Q4; prepare SEO landing page content
- Risk: v1.1 scope creep from flood of user feedback | Mitigation: enforce 6-week freeze on major new features after launch; fix-only sprint first

---

## Dependency Map

```
Q2 2026 → Q3 2026: Auth + persistence layer must be stable before recurring tasks and dashboard build
Q3 2026 → Q4 2026: All 6 acceptance criteria must pass in staging before QA / beta begins
Q4 2026 → Q1 2027: Beta metrics (≥80% on-time completion) must be validated before public launch commitment
```

---

## Risk Register

| Quarter | Risk | Likelihood | Impact | Mitigation |
|---------|------|------------|--------|------------|
| Q2 2026 | Architecture decision stalls development | High | High | Timebox to 1 week; default to local-first |
| Q2 2026 | Auth complexity underestimated | Medium | Medium | Evaluate third-party auth providers |
| Q3 2026 | Recurring task edge cases cause regressions | Medium | High | Dedicated QA sprint for recurrence logic |
| Q3 2026 | Dashboard UX requires excessive iteration | Medium | Medium | Figma prototype + 3-user validation before build |
| Q4 2026 | Beta users expect bank sync and churn | High | Medium | Clear onboarding scope messaging |
| Q4 2026 | PWA performance issues hard to fix late | Low | High | Run Lighthouse audits from Q3 onward |
| Q1 2027 | Low post-launch discoverability | Medium | High | Plan launch channels in Q4 |
| Q1 2027 | v1.1 scope creep from user feedback | High | Medium | 6-week feature freeze post-launch |

---

## Open Questions

- [ ] **By end of April 2026:** Local-first vs. cloud backend — which architecture? (blocks Q2 storage work)
- [ ] **By end of April 2026:** UI component library selection — Shadcn/UI, Radix, MUI, or custom?
- [ ] **By end of Q3 2026:** PWA confirmed as mobile strategy, or pivot to React Native?
- [ ] **By end of Q4 2026:** Will push notifications be included in v1.1 or deferred further?
- [ ] **By end of Q4 2026:** Bank sync (Plaid/Teller) — build in v1.x or remain out of scope indefinitely?
