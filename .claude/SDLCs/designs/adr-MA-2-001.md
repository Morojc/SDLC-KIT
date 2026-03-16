# Architecture Decision Record: Data Persistence Strategy

## ADR Reference
- ADR Number: ADR-001
- Story/Epic Key: MA-2
- Date: 2026-03-16
- Status: Proposed
- Deciders: Tech Lead, Product Lead
- Supersedes: N/A

## Context

### Problem Statement
The app must persist task data across sessions and eventually support optional cloud sync for multi-device access. A storage architecture must be chosen before any data layer code is written (blocks FR-008 and all storage work). The decision must be made by 2026-04-30.

### Constraints
- **Offline-first requirement:** Core read/write task operations must work offline (FR-008, NFR)
- **PWA target:** Service worker + installability already imply client-side asset caching; data must be consistent with this model
- **Single-user MVP:** No multi-user sync complexity in scope
- **Future-proof:** Architecture must not preclude migration to cloud backend post-MVP (PRD explicit constraint)
- **Budget:** MVP should minimize ongoing infrastructure cost (no mandatory backend server)
- **Performance:** Dashboard must render <500ms with 100+ tasks; task CRUD <300ms perceived latency

### Quality Attributes at Stake
| Attribute | Priority | Rationale |
|-----------|----------|-----------|
| Offline availability | Critical | PRD requires offline create/complete; PWA cannot rely on network |
| Performance | High | <300ms CRUD, <500ms dashboard — local reads are inherently faster than network calls |
| Maintainability | High | MVP team is small; avoiding backend ops overhead matters |
| Scalability | Medium | MVP handles ≥1,000 tasks/user; cloud migration is post-MVP |
| Security | High | User data isolation; auth tokens never in localStorage |

## Decision

### Chosen Approach
**Local-first with IndexedDB + migration-ready sync interface**

Use IndexedDB (via [Dexie.js](https://dexie.org/)) as the primary data store in the browser. All task CRUD operations read/write directly to IndexedDB. Design the data access layer (DAL) behind a repository interface so it can be swapped for a cloud-backed implementation post-MVP without touching application logic.

Structure:
```
src/
  data/
    db.ts              # Dexie schema + migrations
    tasks.repository.ts  # ITaskRepository interface
    tasks.local.ts     # IndexedDB implementation
    tasks.sync.ts      # (stub) future cloud sync adapter
```

The service worker caches the app shell; IndexedDB holds task data. Offline writes are immediately committed to IndexedDB and become the source of truth. A future sync layer will handle conflict resolution when cloud backend is added.

### Rationale
- Offline is a hard requirement — local-first satisfies it without complexity
- No backend to provision, deploy, or pay for in MVP
- Dexie.js provides a clean Promise-based API over raw IndexedDB, reducing boilerplate
- Repository pattern isolates the cloud migration path cleanly
- ≥1,000 tasks in IndexedDB is well within browser limits (~50–500 MB quota, tasks are tiny)

## Alternatives Considered

### Alternative 1: Cloud-only REST API + PostgreSQL
**Description:** All data stored server-side; client makes HTTP calls for every CRUD operation.
**Pros:**
- Multi-device sync is native — no future migration needed
- Easier server-side querying, reporting, and analytics
- Centralized backup and data durability
**Cons:**
- Offline support requires complex request queuing (background sync API) with conflict resolution
- Requires standing up and paying for backend infrastructure from day one
- Increases latency for every operation (network round-trip vs. local read)
- Adds authentication surface area earlier
**Why rejected:** Offline is a hard MVP requirement. Building robust offline + sync for a cloud-primary architecture is significantly more complex than local-first. Cost and operational overhead not justified at MVP scale.

### Alternative 2: SQLite via WebAssembly (wa-sqlite / sql.js)
**Description:** Embed SQLite compiled to WASM; run SQL queries client-side.
**Pros:**
- Familiar SQL query model
- Full relational query capability
- Portable: same schema can run server-side if migrated
**Cons:**
- WASM bundle adds ~1–3 MB to initial load (conflicts with Lighthouse ≥90 target)
- Persistence requires additional layer (OPFS or IndexedDB for WASM storage) — adds complexity
- Ecosystem less mature in browser context; debugging harder
- Slower cold start vs. native IndexedDB
**Why rejected:** Bundle size penalty threatens the <2s load performance target. Complexity overhead exceeds benefit for a simple task data model (no complex joins needed).

### Alternative 3: localStorage only
**Description:** Serialize task array to JSON and store in localStorage.
**Pros:**
- Simplest possible implementation
- No library required
**Cons:**
- 5–10 MB storage limit — fails the ≥1,000 tasks NFR at scale
- Synchronous API blocks the main thread
- No indexing — filtering/sorting requires full-scan deserialize of entire dataset
- PRD explicitly prohibits auth tokens in localStorage; mixing data + auth concerns is risky
**Why rejected:** Storage limit and performance characteristics are incompatible with PRD requirements.

## Consequences

### Positive
- Zero infrastructure cost for MVP
- All reads/writes are synchronous-equivalent via IndexedDB (no loading spinners for CRUD)
- Offline mode works out of the box — no queuing logic needed for MVP
- Repository interface makes cloud migration a targeted, isolated change

### Negative / Trade-offs
- Multi-device sync is not possible until cloud backend is added — acceptable, as it's explicitly post-MVP scope
- Data lives only in the user's browser — if they clear storage, data is lost (mitigated by: export feature in backlog, clear user communication)
- IndexedDB has no built-in encryption — acceptable for MVP; sensitive data (amounts) are low-risk without bank credentials present

### Risks
- Risk: User clears browser data and loses all tasks | Mitigation: Add manual export-to-JSON in v1.1; document clearly in onboarding
- Risk: IndexedDB storage quota exceeded on some iOS Safari versions (~50 MB limit) | Mitigation: Monitor task payload size; warn user at 500 tasks if approaching limit
- Risk: Cloud migration requires non-trivial sync conflict resolution design | Mitigation: Define sync interface contracts now; document conflict policy before cloud work begins

## Implementation Notes

### What Changes
- `src/data/db.ts` — Dexie schema with tasks table (id, title, amount, dueDate, category, priority, status, recurrence, completedAt, createdAt)
- `src/data/tasks.repository.ts` — ITaskRepository interface (getAll, getById, create, update, delete, getByStatus, getByCategory)
- `src/data/tasks.local.ts` — Dexie-backed implementation
- `src/data/tasks.sync.ts` — Stub adapter (throws NotImplemented) for future cloud sync

### Migration Path
When cloud backend is added:
1. Implement `tasks.cloud.ts` satisfying ITaskRepository
2. Add background sync: on app start, if authenticated + online, push local delta to server
3. Conflict resolution policy: last-write-wins for MVP cloud sync (document this as known limitation)

### Validation
- Metric: Task create/update latency (measured via Performance API)
- Target: p95 < 300ms for all CRUD operations
- Metric: Dashboard render time with 100+ tasks
- Target: < 500ms (Lighthouse CI gate)
- Review Date: 2026-06-30 — revisit if user data loss incidents reported or multi-device demand validated

## References
- PRD Open Question: "Architecture decision — local-first vs. cloud backend" | Due: 2026-04-30
- [Dexie.js documentation](https://dexie.org/)
- Jira Epic: MA-2
