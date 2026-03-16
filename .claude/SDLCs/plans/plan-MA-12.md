# Implementation Plan: MA-12 — User Authentication and Identity

## Jira Context
- Story Key: MA-12
- Epic: MA-2
- Sprint: Sprint 1 — Foundation (2026-04-01 → 2026-04-14)
- Story Points: 5
- Priority: Critical (Highest)
- Blocks: MA-3, MA-5, MA-7, MA-9, MA-10

## Story
As a user, I want to create an account and log in securely so that my financial tasks are private and accessible only to me.

## Acceptance Criteria
- [ ] New user can sign up with email + password → auto logged in
- [ ] Returning user with valid credentials → authenticated + redirected to dashboard
- [ ] Invalid credentials show generic error: "Invalid email or password" (no field-specific hint)
- [ ] Session persists across browser close/reopen (httpOnly cookie — never localStorage)
- [ ] Session expiry → redirect to /sign-in without data loss
- [ ] Log out invalidates session server-side; local tokens cleared

## Implementation Phases

| Phase | Task | Status | Depends On | Parallel? | Estimate |
|-------|------|--------|------------|-----------|----------|
| 1 | Scaffold Next.js 14 App Router project + install all core dependencies | pending | — | No | 1h |
| 2 | Configure Clerk: env vars, ClerkProvider in layout, publishable key | pending | Phase 1 | No | 1h |
| 3 | Build sign-in and sign-up pages with Clerk embedded components | pending | Phase 2 | No | 1h |
| 4 | Implement middleware: route protection + onboarding redirect logic | pending | Phase 2 | Yes (w/ P3) | 1h |
| 5 | Root redirect, UserButton (logout), session persistence smoke test | pending | Phase 3, 4 | No | 1h |
| 6 | Tests: middleware unit tests + Playwright E2E for auth flows | pending | Phase 5 | No | 2h |

**Total estimate: 7h**

## File Targets

### New files to create
- `finance-tasks/` — Next.js 14 app root (created via `create-next-app`)
- `finance-tasks/middleware.ts` — Clerk `authMiddleware` protecting all routes; redirects unauthenticated users to `/sign-in`; redirects new users (no `onboardingCompleted`) to `/onboarding`
- `finance-tasks/app/layout.tsx` — Root layout with `<ClerkProvider>`; wraps all routes
- `finance-tasks/app/page.tsx` — Root redirect: authed → `/dashboard`, unauthed → `/sign-in`
- `finance-tasks/app/(auth)/sign-in/[[...sign-in]]/page.tsx` — Clerk `<SignIn />` embed, centered layout
- `finance-tasks/app/(auth)/sign-up/[[...sign-up]]/page.tsx` — Clerk `<SignUp />` embed, centered layout
- `finance-tasks/app/(auth)/layout.tsx` — Auth page shell (centered card, no nav)
- `finance-tasks/app/(app)/layout.tsx` — App shell with top nav (includes `<UserButton />`)
- `finance-tasks/app/(app)/dashboard/page.tsx` — Placeholder (returns "Dashboard — coming soon")
- `finance-tasks/lib/auth.ts` — Re-exports `currentUser`, `auth` from `@clerk/nextjs/server`
- `finance-tasks/.env.local` — `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY` (never committed)
- `finance-tasks/.env.example` — Template with placeholder values (committed)
- `finance-tasks/e2e/auth.spec.ts` — Playwright: sign-up, sign-in, invalid creds, logout, session persistence

### Modified files
- `finance-tasks/package.json` — Add all dependencies listed below
- `finance-tasks/next.config.ts` — Enable `@clerk/nextjs` + future `next-pwa` config placeholder
- `finance-tasks/tailwind.config.ts` — shadcn/ui content paths
- `finance-tasks/.gitignore` — Add `.env.local`

## Dependencies to Install

```bash
# Core
npx create-next-app@latest finance-tasks \
  --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*"

cd finance-tasks

# Auth
npm install @clerk/nextjs

# Storage (installed now, wired in MA-10)
npm install dexie

# Data fetching
npm install @tanstack/react-query @tanstack/react-query-devtools

# Date logic
npm install date-fns

# Validation
npm install zod

# UI
npm install class-variance-authority clsx tailwind-merge lucide-react
npx shadcn@latest init --defaults
npx shadcn@latest add button input card badge separator

# Testing
npm install -D vitest @vitejs/plugin-react @testing-library/react @testing-library/user-event jsdom
npm install -D playwright @playwright/test
npx playwright install chromium
```

## Validation Commands

```bash
# After Phase 1 — project builds
cd finance-tasks && npm run build

# After Phase 2 — Clerk provider loads without error
npm run dev  # visit http://localhost:3000 — no ClerkProvider error in console

# After Phase 3 — auth pages render
npm run dev  # visit /sign-in and /sign-up — Clerk embeds visible

# After Phase 4 — middleware protects routes
# Visit /dashboard without signing in → should redirect to /sign-in

# After Phase 5 — full auth flow
# Manual: sign up → lands on /dashboard; sign out → back to /sign-in

# After Phase 6 — automated tests pass
npx playwright test e2e/auth.spec.ts
npm run typecheck  # tsc --noEmit
npm run lint       # eslint . --ext .ts,.tsx
npm run build      # full production build
```

## Playwright E2E Test Plan (`e2e/auth.spec.ts`)

```typescript
test('new user can sign up and reach dashboard')
test('returning user can sign in with valid credentials')
test('invalid credentials show generic error message')
test('session persists after tab close and reopen')
test('session expiry redirects to /sign-in')
test('logout invalidates session and redirects to /sign-in')
test('unauthenticated visit to /dashboard redirects to /sign-in')
test('auth token is NOT present in localStorage or sessionStorage')
```

## Security Checklist
- [ ] `CLERK_SECRET_KEY` is in `.env.local` only — not in `.env` or committed
- [ ] `.env.local` is in `.gitignore`
- [ ] No auth data written to `localStorage` or `sessionStorage` (verify in E2E test)
- [ ] Clerk session cookie flags: `httpOnly: true`, `SameSite: Strict`, `Secure: true` (verify in browser devtools on staging)
- [ ] Generic error message on invalid credentials (no "email not found" vs "wrong password" distinction)

## Jira Sub-tasks
| Sub-task Key | Title | Estimate |
|--------------|-------|----------|
| (TBD) | Phase 1: Scaffold Next.js 14 app + install dependencies | 1h |
| (TBD) | Phase 2: Configure Clerk (env, ClerkProvider, publishable key) | 1h |
| (TBD) | Phase 3: Sign-in and sign-up pages with Clerk embeds | 1h |
| (TBD) | Phase 4: Middleware — route protection + redirect logic | 1h |
| (TBD) | Phase 5: Root redirect, UserButton (logout), smoke test | 1h |
| (TBD) | Phase 6: Tests — middleware unit + Playwright E2E auth flows | 2h |

## Definition of Done
- [ ] All acceptance criteria verified (manual walkthrough + E2E)
- [ ] Playwright E2E tests pass (8 scenarios)
- [ ] TypeScript: `tsc --noEmit` exits 0
- [ ] ESLint: no errors or warnings
- [ ] `npm run build` succeeds (production build)
- [ ] No auth token in localStorage (E2E assertion)
- [ ] `.env.local` not committed; `.env.example` committed
- [ ] Jira MA-12 transitioned to Done
- [ ] MA-10 (persistence) and MA-3 (create task) can now begin in parallel
