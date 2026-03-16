# Architecture Decision Record: UI Component Library Selection

## ADR Reference
- ADR Number: ADR-002
- Story/Epic Key: MA-2
- Date: 2026-03-16
- Status: Proposed
- Deciders: Frontend Lead, Tech Lead
- Supersedes: N/A

## Context

### Problem Statement
A UI component library must be selected before UI development begins. The choice establishes the component API, accessibility baseline, styling approach, and bundle size profile for the entire frontend. Must be decided by 2026-04-30 (blocks all UI work).

### Constraints
- **WCAG AA compliance required:** Keyboard navigation, screen reader support, color contrast (PRD NFR)
- **Performance:** Lighthouse score ≥90; initial load < 2s on mid-range mobile — bundle size matters
- **PWA / mobile-first:** Components must render well on small screens without a native app
- **Small team:** DX and documentation quality reduce onboarding time
- **No IE support:** Modern browsers only (Chrome 110+, Firefox 115+, Safari 16+)
- **Minimum 16px body text; zoom must not break layout**

### Quality Attributes at Stake
| Attribute | Priority | Rationale |
|-----------|----------|-----------|
| Accessibility | Critical | WCAG AA is a hard NFR; library must not require manual ARIA remediation on every component |
| Performance | High | Bundle size directly impacts Lighthouse score and load time target |
| Maintainability | High | Library must be actively maintained; breaking changes block all UI work |
| Developer experience | High | Small team needs good docs and predictable API |
| Customizability | Medium | Finance app needs a clean, focused visual style — not a generic look |

## Decision

### Chosen Approach
**shadcn/ui (Radix UI primitives + Tailwind CSS)**

shadcn/ui is not a traditional component library — it provides copy-paste React components built on [Radix UI](https://www.radix-ui.com/) headless primitives, styled with Tailwind CSS. Components are owned by the project (copied into `src/components/ui/`), not imported from a package.

Structure:
```
src/
  components/
    ui/              # shadcn/ui components (Button, Dialog, Select, etc.)
    finance/         # App-specific composed components (TaskCard, DashboardSection, etc.)
```

Tailwind CSS handles all styling; Radix UI provides fully accessible, unstyled behavior primitives (focus management, keyboard nav, ARIA roles).

### Rationale
- **Accessibility by default:** Radix UI primitives implement WAI-ARIA patterns correctly for every interactive component (Dialog, Select, Popover, etc.) — no manual ARIA work
- **Zero runtime bundle for unopened components:** Since components are copied in rather than imported from a monolithic package, tree-shaking is trivially effective — only used components ship
- **Full design control:** Tailwind classes are directly editable; no fighting against a design system's opinions
- **Active ecosystem:** shadcn/ui is the most-starred React component project as of 2026; Radix UI has Mozilla + Vercel backing
- **TypeScript-first:** All components ship with complete type definitions
- **Tailwind 4 compatible:** No anticipated breaking migration in Q2–Q4 2026 timeline

## Alternatives Considered

### Alternative 1: Material UI (MUI v6)
**Description:** Google Material Design component library for React with a large component catalogue.
**Pros:**
- Extremely comprehensive component set
- Strong accessibility support
- Large community, abundant Stack Overflow answers
- Built-in theming system (MUI Theme)
**Cons:**
- Bundle size: MUI core is ~300 KB min+gzip even with tree-shaking — threatens Lighthouse ≥90 target on mobile
- Strong opinionated Material Design aesthetic requires significant theme overrides to achieve a custom finance-app feel
- Emotion CSS-in-JS runtime adds ~8 KB and runtime style injection overhead
- Component API is verbose (sx prop, styled(), ThemeProvider nesting)
**Why rejected:** Bundle size and runtime CSS-in-JS overhead conflict with the <2s load / Lighthouse ≥90 target on mid-range mobile. Visual customization effort exceeds shadcn/ui approach.

### Alternative 2: Chakra UI v3
**Description:** Component library focused on simplicity and accessibility, styled with CSS variables.
**Pros:**
- Strong accessibility baseline
- Good DX with composable prop API
- v3 dropped Emotion dependency, improving runtime performance
**Cons:**
- Still imports a full component bundle — less granular than shadcn/ui copy-paste model
- Design language is opinionated (Chakra aesthetic is recognizable)
- Smaller community than MUI or shadcn/ui; fewer examples for complex patterns
- v3 migration broke significant v2 API — risk of future churn
**Why rejected:** Bundle granularity and community size are weaker than shadcn/ui for the same accessibility benefit. Chakra's aesthetic requires similar override effort to MUI.

### Alternative 3: Headless UI (Tailwind Labs) + custom components
**Description:** Use Tailwind Labs' Headless UI for a small set of interactive primitives (Dialog, Listbox, Combobox) and build all other components from scratch with Tailwind.
**Pros:**
- Smallest possible bundle (only primitives used ship)
- Maximum design freedom
- No dependency on a component library roadmap
**Cons:**
- Headless UI covers only ~8 components; everything else is hand-rolled
- Building accessible components (date pickers, tooltips, command menus) from scratch is expensive and error-prone
- WCAG AA compliance requires deep expertise — high risk of accessibility regressions without automated a11y testing coverage
**Why rejected:** Engineering cost to build and maintain a fully accessible component set from scratch is disproportionate for a small MVP team. shadcn/ui provides the same bundle advantage with Radix's battle-tested accessibility primitives.

## Consequences

### Positive
- Lighthouse score target achievable — only used component code ships
- WCAG AA baseline met out of the box for all interactive patterns
- Full design control — finance-specific visual language can be implemented without fighting the library
- Components are in the project — no breaking change risk from library version bumps

### Negative / Trade-offs
- Components must be manually updated when shadcn/ui releases improvements — acceptable; updates are opt-in and components are in our codebase
- Less auto-generated component catalogue than MUI — developers must compose patterns explicitly (acceptable trade-off for performance + design control)
- Tailwind CSS class verbosity in JSX — mitigated by extracting reusable `cn()` utility and component variants via `cva`

### Risks
- Risk: Radix UI releases a breaking API change | Mitigation: Pin Radix package versions; upgrade deliberately before major features
- Risk: Tailwind CSS v4 migration breaks utility classes | Mitigation: Run Tailwind upgrade codemods in a dedicated PR; validate with visual regression snapshots
- Risk: Team unfamiliar with Radix primitive composition model | Mitigation: Add a Storybook with all ui/ components documented in sprint 1

## Implementation Notes

### What Changes
- Add `tailwindcss`, `@radix-ui/*`, `class-variance-authority`, `clsx`, `tailwind-merge` to dependencies
- Run `npx shadcn@latest init` to scaffold `src/components/ui/` and `tailwind.config.ts`
- Add needed components: Button, Input, Select, Dialog, Card, Badge, Separator, Popover, Calendar (for date picker)
- Create `src/lib/utils.ts` with `cn()` helper

### Migration Path
No migration needed — this is a greenfield selection. If a component must be replaced later, shadcn/ui components are isolated in `src/components/ui/` and can be swapped component-by-component.

### Validation
- Metric: Lighthouse Performance score on homepage (mobile simulation)
- Target: ≥90
- Metric: Axe-core accessibility violations on all screens
- Target: 0 critical or serious violations
- Review Date: 2026-07-01 — revisit if accessibility audit finds systematic gaps in Radix primitives

## References
- PRD Open Question: "UI component library selection" | Due: 2026-04-30
- [shadcn/ui](https://ui.shadcn.com/)
- [Radix UI](https://www.radix-ui.com/)
- Jira Epic: MA-2
