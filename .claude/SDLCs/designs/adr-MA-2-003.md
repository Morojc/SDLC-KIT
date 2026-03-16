# Architecture Decision Record: Authentication Strategy

## ADR Reference
- ADR Number: ADR-003
- Story/Epic Key: MA-2
- Date: 2026-03-16
- Status: Proposed
- Deciders: Tech Lead, Product Lead
- Supersedes: N/A

## Context

### Problem Statement
The app requires user identity for data isolation (NFR: no user can access another user's data) and to enable future multi-device sync. An auth strategy must be chosen before any user-specific storage or session handling is implemented. Must be decided by 2026-04-30 (blocks FR-008 user isolation and session management).

### Constraints
- **Auth tokens must be in httpOnly cookies or secure storage — never localStorage** (PRD NFR, hard security constraint)
- **Third-party auth preferred over hand-rolled** (PRD explicit guidance — reduce security surface area)
- **Single-user MVP:** No org/team/roles in scope; just individual account login
- **Web-first PWA:** Must work in browsers; iOS Safari httpOnly cookie support required
- **Timeline:** MVP auth must be functional by end of Q2 2026 (foundation milestone)
- **No external integrations:** Auth provider must not require bank credential handling

### Quality Attributes at Stake
| Attribute | Priority | Rationale |
|-----------|----------|-----------|
| Security | Critical | Auth tokens in httpOnly cookies; XSS prevention; user data isolation are hard NFRs |
| Developer velocity | High | Small team; hand-rolling auth is high-risk and time-consuming |
| Maintainability | High | Auth security vulnerabilities are maintained by provider, not our team |
| Scalability | Low | Single-user MVP; auth system doesn't need to handle high concurrency |
| Cost | Medium | MVP budget conscious; per-MAU pricing must stay near zero at early stage |

## Decision

### Chosen Approach
**Clerk — third-party managed authentication**

Use [Clerk](https://clerk.com/) as the authentication provider. Clerk handles session management, token storage (httpOnly cookies), user identity, and the auth UI (sign-in/sign-up embeds). The app integrates via Clerk's React SDK (`@clerk/nextjs` or `@clerk/react`).

Integration points:
```
src/
  middleware.ts        # Clerk auth middleware (Next.js) — protects all routes
  app/
    sign-in/[[...sign-in]]/page.tsx   # Clerk <SignIn /> embed
    sign-up/[[...sign-up]]/page.tsx   # Clerk <SignUp /> embed
  lib/
    auth.ts            # Re-export currentUser(), auth() helpers
```

All task data in IndexedDB is keyed by `userId` (Clerk user ID). User can only query tasks where `userId === currentUser.id`.

### Rationale
- **httpOnly cookie sessions out of the box:** Clerk uses secure, httpOnly cookies for session tokens — satisfies the PRD security NFR without any custom implementation
- **Generous free tier:** Clerk's free plan supports 10,000 MAU — more than sufficient for MVP and beta
- **React SDK quality:** Pre-built `<SignIn />`, `<SignUp />`, `<UserButton />` components with correct ARIA; no need to build auth UI from scratch
- **JWKS validation:** Clerk issues JWTs verifiable via JWKS endpoint — future API routes can validate tokens without calling Clerk on every request
- **Social login ready:** Google/GitHub social sign-in can be enabled with one click in Clerk dashboard — useful for future growth without code changes
- **SOC 2 Type II certified:** Security posture appropriate for handling user identity in a finance-adjacent app

## Alternatives Considered

### Alternative 1: Auth.js (NextAuth v5)
**Description:** Open-source authentication library for Next.js; handles sessions, OAuth providers, and database adapters.
**Pros:**
- Self-hosted — no dependency on third-party service
- Free at any scale
- Supports most OAuth providers (Google, GitHub, etc.)
- Active community; well-documented for Next.js
**Cons:**
- Session token storage requires explicit configuration for httpOnly cookies — default setup needs review to meet PRD NFR
- No pre-built UI — must build sign-in/sign-up forms from scratch (accessible, validated forms are non-trivial)
- Database adapter required for persistent sessions if using local-first model — adds complexity
- Security configuration surface area: CSRF, session rotation, secure cookie flags all need explicit review
**Why rejected:** Auth.js is powerful but shifts significant security configuration responsibility to our team. Given the PRD explicitly prefers third-party auth to reduce surface area, and MVP team is small, Auth.js adds risk without proportional benefit at this scale.

### Alternative 2: Auth0
**Description:** Enterprise-grade identity platform (Okta subsidiary) with broad protocol support.
**Pros:**
- Extremely comprehensive: SAML, OIDC, MFA, anomaly detection, compliance features
- Large enterprise adoption; excellent security track record
- Managed UI (Universal Login) handles all auth flows
**Cons:**
- Free tier is 7,500 MAU — similar to Clerk but with a more complex dashboard and higher paid tier pricing
- Configuration complexity is tuned for enterprise (organizations, roles, actions pipeline) — overkill for a single-user consumer app
- Universal Login redirect flow breaks PWA UX (redirects out of the PWA shell, breaking standalone mode)
- SDK for React is heavier than Clerk's; slower to integrate
**Why rejected:** Auth0's redirect-based Universal Login disrupts PWA standalone mode experience. Configuration complexity is disproportionate to a single-user consumer app. Clerk provides equivalent security with significantly better PWA-compatible embedded UI.

### Alternative 3: Custom JWT auth (hand-rolled)
**Description:** Implement email/password auth with bcrypt hashing, JWT issuance, and httpOnly cookie session management in a custom API backend.
**Pros:**
- Zero third-party dependency
- Full control over token lifetime, refresh strategy, and storage
- No per-MAU cost at scale
**Cons:**
- Requires standing up a backend (Node/Express or Next.js API routes) before any feature work — delays Foundation milestone
- Bcrypt, JWT, CSRF, session rotation, rate limiting, account lockout — each a potential vulnerability if misconfigured
- Password reset flow, email verification, and account recovery are non-trivial to build correctly
- PRD explicitly calls this out as the anti-pattern to avoid ("third-party auth preferred over hand-rolled to reduce surface area")
**Why rejected:** The PRD explicitly disprefers this approach. The security risk and development time cost of hand-rolling auth correctly is not justified when managed providers offer equivalent security at zero cost for MVP scale.

## Consequences

### Positive
- httpOnly cookie session security satisfied without custom code
- Auth UI (sign-in, sign-up, profile) ships in days not weeks
- Vulnerability patching in auth layer is Clerk's responsibility
- Social login (Google) can be enabled post-MVP with zero code changes
- JWKS-based token validation keeps future API routes stateless

### Negative / Trade-offs
- Dependency on Clerk's availability — if Clerk has an outage, users cannot log in (mitigated: tasks are readable offline from IndexedDB even if auth is unavailable for new logins)
- Vendor lock-in: migrating away from Clerk requires re-implementing session management and migrating user IDs — acceptable at MVP; revisit if scale demands it
- Free tier limits (10K MAU): if MVP goes viral, Clerk costs begin at ~$25/month for 10K+ MAU — acceptable and a good problem to have

### Risks
- Risk: Clerk's free tier policy changes | Mitigation: Architecture uses Clerk's JWT/user-ID contract only; switching to Auth.js later is isolated to auth middleware and sign-in pages
- Risk: Clerk SDK updates break sign-in flow | Mitigation: Pin Clerk SDK version; upgrade in dedicated auth PR with full sign-in/sign-up regression test
- Risk: httpOnly cookies blocked in cross-origin iframe contexts (if app is ever embedded) | Mitigation: N/A for MVP — app is a standalone PWA, not embedded

## Implementation Notes

### What Changes
- Add `@clerk/nextjs` (or `@clerk/react` if not using Next.js) to dependencies
- Set `CLERK_PUBLISHABLE_KEY` and `CLERK_SECRET_KEY` in `.env.local` (never committed)
- Add `middleware.ts` at project root to protect all routes except `/sign-in`, `/sign-up`
- All IndexedDB task queries must filter by `userId: clerk.user.id`
- Sign-in and sign-up pages use `<SignIn />` and `<SignUp />` embeds (Clerk-managed UI)

### Migration Path
If migrating away from Clerk post-MVP:
1. Export all Clerk user IDs and emails via Clerk dashboard
2. Implement replacement auth (Auth.js + adapter) with same user-ID schema
3. Map old Clerk user IDs to new system IDs in a migration script
4. Update `middleware.ts` and `auth.ts` — no changes to task data layer (userId field is portable)

### Validation
- Metric: Auth flow completion rate (sign-up → first task created)
- Target: ≥90% of users who start sign-up complete it within 5 minutes
- Metric: Zero localStorage auth token storage (verified via browser devtools audit in CI)
- Target: 0 auth-related tokens in localStorage or sessionStorage
- Review Date: 2026-07-01 — revisit if Clerk pricing becomes a concern or if MAU approaches free tier limit

## References
- PRD Open Question: "Auth strategy — third-party (Clerk/Auth0) vs. custom" | Due: 2026-04-30
- PRD NFR: "Auth tokens stored in httpOnly cookies or secure storage (never localStorage)"
- [Clerk documentation](https://clerk.com/docs)
- Jira Epic: MA-2
