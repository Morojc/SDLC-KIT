# Technical Design: Personal Finance Task Tracker MVP

## Design Reference
- Epic Key: MA-2
- Stories: MA-3 through MA-12
- Created: 2026-03-16
- Status: Draft
- Reviewer: TBD
- ADRs: ADR-001 (IndexedDB local-first), ADR-002 (shadcn/ui), ADR-003 (Clerk auth)

## Overview
A Next.js 14 (App Router) PWA that lets users create, track, and complete financial tasks. Data is persisted locally via IndexedDB (Dexie.js), auth is managed by Clerk, and the UI is built with shadcn/ui (Radix UI + Tailwind CSS). The architecture uses a repository pattern so the storage layer can be swapped for a cloud backend post-MVP without touching application logic.

---

## Tech Stack

| Concern | Choice | Rationale |
|---------|--------|-----------|
| Framework | Next.js 14 (App Router) | SSR + RSC; `@clerk/nextjs` middleware; `next-pwa` for service worker |
| Language | TypeScript 5 | Type safety across data models, API contracts, and Clerk user types |
| Auth | Clerk (`@clerk/nextjs`) | httpOnly cookie sessions; embedded SignIn/SignUp; JWKS token validation |
| Storage | IndexedDB via Dexie.js 4 | Local-first, offline-capable; repository interface for future cloud swap |
| UI | shadcn/ui + Radix UI + Tailwind CSS 4 | WCAG AA baseline; zero-bundle for unused components |
| Date logic | date-fns v3 | Recurrence calculations, end-of-month normalization, timezone-aware comparisons |
| State | TanStack Query v5 | Async state + cache invalidation for IndexedDB reads; no global store needed |
| PWA | next-pwa | Service worker generation; cache-first app shell; manifest.json |
| Validation | Zod | Schema validation at form submission and repository boundaries |
| Testing | Vitest + Testing Library + Playwright | Unit, component, and E2E; Dexie fake for unit tests |

---

## Data Models

### Task
Primary entity. All tasks belong to an authenticated user.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | string (ULID) | PK, NOT NULL | Unique task ID (ULID for natural sort order) |
| userId | string | NOT NULL, indexed | Clerk user ID — all queries filter by this |
| title | string | NOT NULL, max 200 | Task display name |
| amount | number | NOT NULL, ≥0, 2dp | Financial amount in user's local currency |
| dueDate | string (ISO date) | NOT NULL | `YYYY-MM-DD` — stored as date string, not timestamp |
| category | TaskCategory | NOT NULL | Enum: `bills \| savings \| debt \| income \| other` |
| priority | TaskPriority | NOT NULL | Enum: `high \| medium \| low` |
| status | TaskStatus | NOT NULL | Enum: `active \| completed` |
| recurrence | RecurrenceRule \| null | nullable | Null = one-off task |
| completedAt | string (ISO datetime) \| null | nullable | UTC ISO string when completed |
| onTime | boolean \| null | nullable | Derived on completion: `completedAt date ≤ dueDate` |
| createdAt | string (ISO datetime) | NOT NULL | UTC ISO string at creation |
| updatedAt | string (ISO datetime) | NOT NULL | UTC ISO string at last modification |

**Indexes:** `userId`, `[userId+status]`, `[userId+category]`, `[userId+dueDate]`

### RecurrenceRule
Embedded in Task (stored as JSON in IndexedDB, not a separate table).

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| frequency | RecurrenceFrequency | NOT NULL | Enum: `weekly \| biweekly \| monthly \| quarterly \| custom` |
| intervalDays | number \| null | Required if `custom` | Number of days between occurrences |
| seriesId | string (ULID) | NOT NULL | Groups all occurrences of a recurring series |
| originalDueDate | string (ISO date) | NOT NULL | Anchor date for drift-free scheduling |

**Recurrence calculation rule:** Next due date is always calculated from `originalDueDate + (completionCount * interval)` — never from `completedAt`. This prevents schedule drift.

### UserPreferences
Stored in a separate Dexie table. One row per user.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| userId | string | PK | Clerk user ID |
| onboardingCompleted | boolean | NOT NULL | Prevents re-showing onboarding |
| defaultCategory | TaskCategory \| null | nullable | Pre-selected category in task form |
| lastFilterCategory | TaskCategory \| null | nullable | Persisted within session via URL param (not here) |
| createdAt | string | NOT NULL | |

---

## Repository Interface

```typescript
// src/data/tasks.repository.ts
export interface ITaskRepository {
  getAll(userId: string): Promise<Task[]>;
  getById(userId: string, id: string): Promise<Task | undefined>;
  getByStatus(userId: string, status: TaskStatus): Promise<Task[]>;
  getByCategory(userId: string, category: TaskCategory): Promise<Task[]>;
  getDashboard(userId: string, today: string): Promise<DashboardData>;
  create(task: NewTask): Promise<Task>;
  update(userId: string, id: string, patch: Partial<Task>): Promise<Task>;
  delete(userId: string, id: string): Promise<void>;
  deleteSeriesFuture(userId: string, seriesId: string, fromDate: string): Promise<void>;
  updateSeries(userId: string, seriesId: string, patch: Partial<Task>): Promise<void>;
}

export interface DashboardData {
  overdue: Task[];      // dueDate < today && status === active
  dueToday: Task[];     // dueDate === today && status === active
  upcoming: Task[];     // today < dueDate <= today+7 && status === active
}
```

```typescript
// src/data/tasks.local.ts — Dexie implementation
export class LocalTaskRepository implements ITaskRepository { ... }

// src/data/tasks.sync.ts — future cloud stub
export class CloudTaskRepository implements ITaskRepository {
  // Throws NotImplementedError until cloud backend exists
}
```

### Dexie Schema

```typescript
// src/data/db.ts
class FinanceDB extends Dexie {
  tasks!: Table<Task>;
  userPreferences!: Table<UserPreferences>;

  constructor() {
    super('FinanceTaskDB');
    this.version(1).stores({
      tasks: 'id, userId, [userId+status], [userId+category], [userId+dueDate]',
      userPreferences: 'userId',
    });
  }
}
```

Schema versioning: increment `version(N)` for any structural changes; never mutate existing versions.

---

## API Contracts

This is a local-first PWA with no backend in MVP. "API" refers to the TypeScript repository interface consumed by React components via TanStack Query hooks.

### TanStack Query Hooks

```typescript
// src/hooks/useTasks.ts
useDashboard(userId)          // → DashboardData
useTaskList(userId, filters)  // → Task[]
useTask(userId, id)           // → Task | undefined
useCreateTask()               // → mutation
useUpdateTask()               // → mutation
useDeleteTask()               // → mutation
useCompleteTask()             // → mutation (sets status, completedAt, onTime; triggers recurrence)
useUserPreferences(userId)    // → UserPreferences
useUpdatePreferences()        // → mutation
```

All mutations call `queryClient.invalidateQueries` on success to keep dashboard and task list in sync.

### Future Cloud Endpoints (POST-MVP reference)
When a cloud backend is added, these REST endpoints will be implemented and `CloudTaskRepository` will call them:

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/v1/tasks` | Clerk session | List all tasks for current user |
| POST | `/api/v1/tasks` | Clerk session | Create a task |
| PATCH | `/api/v1/tasks/:id` | Clerk session | Update task fields |
| DELETE | `/api/v1/tasks/:id` | Clerk session | Delete task |
| POST | `/api/v1/tasks/:id/complete` | Clerk session | Complete task + trigger recurrence |
| GET | `/api/v1/tasks/dashboard` | Clerk session | Get bucketed dashboard data |
| GET | `/api/v1/preferences` | Clerk session | Get user preferences |
| PATCH | `/api/v1/preferences` | Clerk session | Update preferences |

---

## UI/UX Flows

### App Shell & Routing

```
/                     → redirect to /dashboard (if authed) or /sign-in
/sign-in              → Clerk <SignIn /> embed
/sign-up              → Clerk <SignUp /> embed
/onboarding           → multi-step onboarding (new users only)
/dashboard            → home: overdue / due today / upcoming sections
/tasks                → full task list with filter + sort controls
/tasks/new            → task creation form
/tasks/[id]/edit      → task edit form (reuses same form component)
```

### Flow 1: New User Onboarding (MA-9)
1. User signs up via Clerk → `onboardingCompleted: false` in UserPreferences
2. Middleware detects `onboardingCompleted === false` → redirect to `/onboarding`
3. Step 1: Welcome screen ("Track your bills and savings — no spreadsheets")
4. Step 2: Template selection — grid of 5+ pre-built templates with checkboxes:
   - Rent (Bills, High priority, monthly)
   - Electricity Bill (Bills, Medium, monthly)
   - Internet Bill (Bills, Medium, monthly)
   - Monthly Savings Transfer (Savings, High, monthly)
   - Credit Card Payment (Debt, High, monthly)
   - + "Add my own" shortcut
5. Selected templates → `useCreateTask()` in batch with due date = end of current month
6. Step 3: Confirmation — "3 tasks added" → CTA "Go to Dashboard"
7. Set `onboardingCompleted: true` → never shown again

### Flow 2: Task Creation (MA-3)
1. User taps "+ New Task" (FAB on dashboard or list)
2. `TaskForm` renders with fields:
   - Title (text input, required)
   - Amount (numeric input, 2dp, locale-formatted)
   - Due Date (date picker via Radix Popover + Calendar)
   - Category (Radix Select: Bills / Savings / Debt / Income / Other)
   - Priority (segmented control: High / Medium / Low)
   - Recurring? (toggle) → if on: Frequency selector (Weekly / Bi-weekly / Monthly / Quarterly / Custom N days)
3. Zod schema validates on submit; inline errors shown per field
4. Past due date shows amber warning badge (not an error block)
5. On success: optimistic insert → navigate back → dashboard/list refreshes via query invalidation

### Flow 3: Task Completion with Undo (MA-5)
1. User taps checkbox / "Complete" on a TaskCard
2. **Delay before persist:** `completeTask()` starts a 5-second timer
3. Toast appears: "Rent marked complete ✓ [Undo]"
4. If user taps Undo within 5s → timer cancelled, toast dismissed, no DB write
5. After 5s → DB write: `status: completed`, `completedAt: now()`, `onTime: completedAt ≤ dueDate`
6. If task is recurring → `generateNextOccurrence()`:
   - Clone task fields
   - Calculate next due date: `addDays(originalDueDate, intervalDays * completionCount)` (or `addMonths` for monthly)
   - End-of-month normalization via `date-fns/lastDayOfMonth`
   - Insert new task with same `seriesId`
7. Query invalidation → task disappears from active dashboard immediately (optimistic)

### Flow 4: Dashboard Rendering (MA-7)
1. `useDashboard(userId)` fetches bucketed tasks from Dexie
2. Date comparison uses user's local timezone: `toLocaleDateString('en-CA')` → `YYYY-MM-DD`
3. Three sections render in order: Overdue (red) → Due Today (amber) → Upcoming (neutral)
4. Empty state per section: only shown when all 3 sections are empty ("You're all caught up! 🎉")
5. Complete action from dashboard card → Flow 3 above; card disappears in real time

### Flow 5: Filter & Sort (MA-8)
1. Filter/sort state stored in URL search params: `?category=bills&sort=dueDate`
2. On mount, read params → apply to `useTaskList(userId, { category, sort })`
3. Category filter: single-select pill group (Bills / Savings / Debt / Income / Other / All)
4. Sort: toggle button group (Due Date ↑ / Priority ↓)
5. Filter + sort applied in Dexie query (not client-side array filter) for performance

---

## Component Architecture

```
src/
  app/
    (auth)/
      sign-in/[[...sign-in]]/page.tsx
      sign-up/[[...sign-up]]/page.tsx
    (app)/
      layout.tsx          # ClerkProvider + TanStack QueryProvider + app shell nav
      dashboard/page.tsx
      tasks/
        page.tsx           # Task list
        new/page.tsx
        [id]/edit/page.tsx
    onboarding/page.tsx
    middleware.ts          # Clerk auth + onboarding redirect
  components/
    ui/                    # shadcn/ui components (Button, Input, Select, Dialog…)
    finance/
      TaskCard.tsx         # Displays a single task; completion checkbox; edit/delete actions
      TaskForm.tsx         # Shared create + edit form with Zod validation
      DashboardSection.tsx # Titled section with task list + empty state
      RecurrenceSelector.tsx
      CategoryFilter.tsx
      SortControl.tsx
      OnboardingTemplates.tsx
      UndoToast.tsx
  data/
    db.ts                  # Dexie schema
    tasks.repository.ts    # ITaskRepository interface
    tasks.local.ts         # Dexie implementation
  hooks/
    useTasks.ts            # TanStack Query hooks wrapping repository
    usePreferences.ts
    useRecurrence.ts       # Next-occurrence calculation logic
  lib/
    utils.ts               # cn(), ULID generator, date helpers
    recurrence.ts          # calculateNextDueDate(), normalizeEndOfMonth()
    templates.ts           # Hardcoded onboarding task templates
    validations.ts         # Zod schemas for Task, RecurrenceRule
  types/
    task.ts                # Task, NewTask, TaskCategory, TaskStatus, RecurrenceRule types
```

---

## PWA Configuration

### manifest.json (via next-pwa)
```json
{
  "name": "Finance Tasks",
  "short_name": "FinTasks",
  "start_url": "/dashboard",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#18181b",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

### Service Worker Caching Strategy
| Resource | Strategy | Rationale |
|----------|----------|-----------|
| App shell (JS/CSS/HTML) | Cache-first | Static assets; versioned by build hash |
| Fonts / icons | Cache-first, 30-day TTL | Never change |
| Clerk auth endpoints | Network-only | Must be fresh; no caching |
| IndexedDB (task data) | N/A — browser-managed | Not a network request |

Offline fallback: if Clerk auth request fails offline → show cached dashboard with read-only badge. Writes are queued in IndexedDB with `pendingSync: true` flag (used when cloud backend is added).

---

## Security Considerations

1. **Auth tokens:** Clerk manages session tokens in httpOnly cookies. No auth data written to localStorage, sessionStorage, or IndexedDB.
2. **User data isolation:** Every Dexie query includes `userId: clerk.user.id` as a filter. The repository interface enforces this — raw table access is never exposed to components.
3. **Input validation:** All user inputs pass through Zod schemas before reaching the repository. Title is trimmed; amount is parsed as a bounded float; category and priority are validated against enums.
4. **XSS prevention:** shadcn/ui renders user-supplied content as text nodes (never `dangerouslySetInnerHTML`). Amount and date fields are type-controlled inputs.
5. **CSRF:** Next.js App Router + Clerk's same-origin cookie configuration mitigates CSRF for authenticated routes. Verify `SameSite=Strict` on Clerk session cookie in production.
6. **Sensitive data in IndexedDB:** Amounts are stored in plaintext. For MVP (no bank credentials, no SSNs), this is acceptable. Document clearly: IndexedDB is not encrypted by default; advise users against storing account numbers or passwords in task titles.

---

## Migration Plan

### Dexie Schema Versioning
- Schema is defined in `db.ts` with explicit `version(N)` blocks
- Version 1 ships with: `tasks` table + `userPreferences` table
- Any future field additions use `version(2).stores({ ... }).upgrade(tx => { ... })`
- Never modify `version(1)` after it ships — only add new version blocks

### Future Cloud Migration Path (post-MVP)
1. Implement `CloudTaskRepository` satisfying `ITaskRepository`
2. Add an export step on first cloud login: read all local Dexie tasks → POST to cloud API
3. Switch DI binding from `LocalTaskRepository` to `CloudTaskRepository` behind a feature flag
4. Conflict resolution: last-write-wins based on `updatedAt` timestamp (document as known limitation)
5. Deprecate local writes once sync is stable; IndexedDB becomes read-cache only

### Feature Flags (MVP)
| Flag | Default | Purpose |
|------|---------|---------|
| `NEXT_PUBLIC_ENABLE_CLOUD_SYNC` | `false` | Gates cloud sync code paths |
| `NEXT_PUBLIC_ONBOARDING_ENABLED` | `true` | Can disable for testing without clearing DB |

---

## Open Design Questions

- [ ] **Amount currency handling:** Store raw number or currency-code + amount pair? If user moves locales, display could be wrong. Recommendation: store `{ amount: number, currencyCode: string }` from day one. Owner: Tech Lead | Due: 2026-04-15
- [ ] **Undo toast for recurring task completion:** If the next occurrence was already generated and the user undoes the completion, delete the generated occurrence too? Recommendation: yes — delete the pending generated task on undo. Owner: Product | Due: 2026-04-15
- [ ] **Recurrence series edit scope:** When a user edits a recurring task and chooses "all future," do we mutate all future Dexie records in the series, or store a delta on the series root? Recommendation: mutate all future records (simpler for local-first) | Due: 2026-04-30
- [ ] **Offline indicator UX:** When offline, show a persistent banner or a subtle status dot? Recommendation: subtle dot in nav to avoid banner blindness | Due: 2026-05-01
- [ ] **Task list pagination:** With 1,000+ tasks, flat list will be slow. Virtual list (TanStack Virtual) or paginated? Recommendation: virtual list for task list; dashboard is already bounded to overdue + today + 7 days | Due: 2026-05-15

---

## Alternatives Considered

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| Next.js App Router | RSC, file-based routing, Clerk middleware native | Learning curve for RSC + client component boundary | **Chosen** |
| Vite + React SPA | Simpler mental model, faster HMR | No SSR; Clerk middleware pattern is Next.js-specific; more PWA config work | Rejected |
| TanStack Router + Vite | Type-safe routing | No SSR; immature compared to Next.js for PWA + auth combo | Rejected |
| Zustand for state | Simple global store | TanStack Query already handles async state; adding Zustand is redundant for this data model | Rejected |
| Dexie React Hooks (built-in) | Less boilerplate | Bypasses repository interface; couples components to Dexie directly — breaks cloud migration path | Rejected |

---

## Story-to-Design Mapping

| Story | Key Components / Hooks | Data Operations |
|-------|------------------------|-----------------|
| MA-3 Create task | `TaskForm`, `useCreateTask` | `repo.create()` |
| MA-4 Edit/delete | `TaskForm` (edit mode), `useUpdateTask`, `useDeleteTask` | `repo.update()`, `repo.delete()`, `repo.deleteSeriesFuture()` |
| MA-5 Complete + undo | `TaskCard`, `UndoToast`, `useCompleteTask` | `repo.update(status, completedAt, onTime)` |
| MA-6 Recurrence | `RecurrenceSelector`, `useRecurrence`, `recurrence.ts` | `repo.create()` (next occurrence) |
| MA-7 Dashboard | `DashboardSection`, `useDashboard` | `repo.getDashboard()` |
| MA-8 Filter/sort | `CategoryFilter`, `SortControl`, `useTaskList` | `repo.getByCategory()` + sort |
| MA-9 Onboarding | `OnboardingTemplates`, `useCreateTask` (batch) | `repo.create()` × N |
| MA-10 Persistence | `db.ts`, `tasks.local.ts` | All Dexie reads/writes |
| MA-11 PWA | `manifest.json`, `next-pwa` config, service worker | Cache-first app shell |
| MA-12 Auth | `middleware.ts`, Clerk `<SignIn />`, `<SignUp />` | Clerk session; userId on all repo calls |
