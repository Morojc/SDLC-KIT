# SDLC KIT — Complete Pipeline Guide

> From a raw idea in your head to a tagged release on `main` — every step, every command, every artifact.

---

## Table of Contents

1. [What is this plugin?](#1-what-is-this-plugin)
2. [How to load it](#2-how-to-load-it)
3. [What happens at session start](#3-what-happens-at-session-start)
4. [The full pipeline](#4-the-full-pipeline)
   - [Phase 0 — Kickoff (one-shot)](#phase-0--kickoff-one-shot)
   - [Phase 1 — Planning](#phase-1--planning)
   - [Phase 2 — Requirements](#phase-2--requirements)
   - [Phase 3 — Design](#phase-3--design)
   - [Phase 4 — Development](#phase-4--development)
   - [Phase 5 — Testing](#phase-5--testing)
   - [Phase 6 — Deployment](#phase-6--deployment)
   - [Phase 7 — Maintenance](#phase-7--maintenance)
5. [Git branching model](#5-git-branching-model)
6. [Jira ↔ Local sync](#6-jira--local-sync)
7. [Background tools: Mulch, Seeds, Overstory](#7-background-tools-mulch-seeds-overstory)
8. [All skills at a glance](#8-all-skills-at-a-glance)
9. [Environment variables required](#9-environment-variables-required)
10. [Folder structure](#10-folder-structure)

---

## 1. What is this plugin?

SDLC KIT is a Claude Code plugin that manages the **complete software development lifecycle** — from a raw idea to a deployed, versioned product — using AI-driven artifact generation, Jira integration, and Git branch management.

**You give it an idea. It gives you:**
- A project brief document (uploaded to GitHub + Confluence)
- A Jira epic, PRD, and full product backlog
- User stories with acceptance criteria (each as a Jira ticket)
- A Sprint 1 plan with capacity estimation
- Git branches for every sprint and every dev story
- Implementation plans, code, tests, commits, and PRs
- Guarded merges that only proceed when Jira status is Done
- Release notes and a tagged version on `main`

**Two MCPs power all external operations:**
- `jira` (mcp-atlassian) — Jira + Confluence
- `github` (@modelcontextprotocol/server-github) — Git operations

**Three background tools run throughout every session:**
- `mulch` — expertise memory (learns your conventions across sessions)
- `seeds` (`sd`) — local issue tracking alongside Jira
- `overstory` — parallel agent spawning for independent tasks

---

## 2. How to Load it

```bash
claude --plugin-dir ./sdlc-launcher
```

All skills become available as `/sdlc-launcher:sdlc-*`.

**Required environment variables** (set before loading):
```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="you@yourcompany.com"
export JIRA_API_TOKEN="your-api-token"
export CONFLUENCE_URL="https://yourcompany.atlassian.net/wiki"
export GITHUB_TOKEN="your-github-pat"
```

---

## 3. What Happens at Session Start

The moment you open a Claude Code session with this plugin loaded, a `SessionStart` hook fires automatically:

```
→ mulch prime --context
  Loads expertise records relevant to the files changed in this project.
  Claude now knows: prior architectural decisions, naming conventions,
  patterns discovered in previous sessions.

→ sd prime
  Loads your open Seeds issues, workflow rules, and command reference.
  Claude now knows: what work is in progress, what is blocked, what is next.
```

You see nothing — it runs silently in the background. But Claude's context is already loaded with project-specific knowledge before you type a single command.

---

## 4. The Full Pipeline

---

### Phase 0 — Kickoff (one-shot)

> **Use this when:** You have a new idea and want to go from 0 to a full Jira backlog in one command.

```bash
/sdlc-launcher:sdlc-kickoff "your idea here"
```

**What happens inside:**

```
STEP 0 — Jira Pull Check
  Claude asks: "Have you made changes in Jira since last session?"
  If yes → runs sdlc-pull first (syncs Jira → local), then continues.

STEP 1 — Clarify & Scope
  Claude asks up to 3 targeted questions to fill gaps:
  - Who is the primary user?
  - Web, mobile, or both?
  - MVP or production-grade?
  Reasonable assumptions are made for everything else.

STEP 2 — Project Brief Generated
  File created: .claude/SDLCs/briefs/brief-{slug}.md
  Contains:
    - Vision & Problem Statement
    - Target Users & Personas
    - Goals & Success Metrics (KPIs)
    - Feature Scope (In / Out of MVP)
    - High-Level Architecture direction
    - Timeline & Phases
    - Risks & Assumptions
    - Stakeholders
  Uploaded to:
    → GitHub: /docs/project-brief-{slug}.md (via github MCP)
    → Confluence: new page linked to epic (via jira MCP)

STEP 3 — Jira Epic Created
  jira MCP: jira_create_issue (type: Epic)
  Returns epic key (e.g. MA-2)
  File created: .claude/SDLCs/epics/epic-MA-2.md

STEP 4 — Overstory Spawns 3 Parallel Agents
  Agent A (branch: overstory/kickoff-prd-MA-2)
    → Generates PRD
    → File: .claude/SDLCs/prds/prd-MA-2.md

  Agent B (branch: overstory/kickoff-stories-MA-2)
    → Generates 8–12 user stories with acceptance criteria
    → Files: .claude/SDLCs/stories/story-MA-2-{N}.md

  Agent C (branch: overstory/kickoff-arch-MA-2)
    → Generates Architecture Decision Records (ADRs)
    → Files: .claude/SDLCs/designs/adr-MA-2-00{N}.md

  All three run simultaneously. Main agent waits for all to finish,
  then merges results.

STEP 5 — Product Backlog Built
  For each story → jira MCP creates a Story ticket
  Printed in terminal:
  ─────────────────────────────────────────────
  MA-3  Create a financial task    [Critical] [3pts] ✓
  MA-4  Edit and delete tasks      [High]     [2pts] ✓
  MA-5  Complete task + timestamp  [Critical] [2pts] ✓
  ...
  ─────────────────────────────────────────────
  Total: 10 stories | 41 points

STEP 6 — Sprint 1 Planned
  Critical-path stories selected (respects dependencies)
  Capacity: 1 dev × 10 days × 0.7 = ~14 story points
  jira MCP: jira_create_sprint → Sprint 1 created
  jira MCP: jira_add_issues_to_sprint → stories assigned

STEP 7 — Git Branches Created
  github MCP: create_branch sprint/1-foundation from main
  github MCP: create_branch story/MA-3-create-task from sprint/1-foundation
  github MCP: create_branch story/MA-5-complete-task from sprint/1-foundation
  ... (one per dev story — UX/research stories are skipped)

STEP 8 — Kickoff Summary Comment on Jira Epic
  jira MCP: jira_add_comment on MA-2
  "Kickoff complete. 10 stories in backlog. Sprint 1 created.
   Brief: {GitHub URL} | {Confluence URL}"

STEP 9 — Session Close
  mulch learn → mulch record → mulch sync
  sd close <kickoff-id> → sd sync
```

**End result:** Jira board fully populated, Git branches ready, project brief published. Zero manual work.

---

### Phase 1 — Planning

> **Use these when:** You want to run planning steps individually instead of all-at-once.

| Skill | What it does | Input | Output |
|-------|-------------|-------|--------|
| `sdlc-idea` | Captures and structures a raw idea | Free text | `.claude/SDLCs/ideas/idea-{N}.md` |
| `sdlc-epic` | Converts idea into a Jira Epic | Idea file or key | `.claude/SDLCs/epics/epic-{KEY}.md` + Jira Epic ticket |
| `sdlc-roadmap` | Generates phased product roadmap | Epic key | `.claude/SDLCs/epics/roadmap-{KEY}.md` + Jira comment |

```bash
/sdlc-launcher:sdlc-idea "personal finance task tracker"
/sdlc-launcher:sdlc-epic idea-001.md
/sdlc-launcher:sdlc-roadmap MA-2
```

---

### Phase 2 — Requirements

| Skill | What it does | Input | Output |
|-------|-------------|-------|--------|
| `sdlc-prd` | Creates full Product Requirements Document | Epic key | `.claude/SDLCs/prds/prd-{KEY}.md` + Jira comment |
| `sdlc-stories` | Breaks PRD into user stories with ACs | Epic key | `.claude/SDLCs/stories/story-{KEY}-{N}.md` × N + Jira Story tickets |
| `sdlc-acceptance` | Writes detailed acceptance criteria | Story key | Updated story file + Jira AC field |

```bash
/sdlc-launcher:sdlc-prd MA-2
/sdlc-launcher:sdlc-stories MA-2
/sdlc-launcher:sdlc-acceptance MA-3
```

**Each story file contains:**
```
- Story Key, Epic Key, Priority, Points, Status
- Type: dev | ux | research | qa    ← determines if a Git branch is created
- Branch: story/{KEY}-{slug}        ← filled when sprint is assigned
- Sprint Branch: sprint/{N}-{slug}  ← filled when sprint is assigned
- User Story (As a / I want / So that)
- Acceptance Criteria (Given/When/Then)
- Technical Notes
- Dependencies
```

---

### Phase 3 — Design

| Skill | What it does | Input | Output |
|-------|-------------|-------|--------|
| `sdlc-arch` | Creates Architecture Decision Records (ADRs) | Epic/story key | `.claude/SDLCs/designs/adr-{KEY}-{N}.md` + Jira comment |
| `sdlc-design` | Creates full technical design document | Epic/story key | `.claude/SDLCs/designs/design-{KEY}.md` + Jira comment |

```bash
/sdlc-launcher:sdlc-arch MA-2
/sdlc-launcher:sdlc-design MA-2
```

**ADR covers:** architectural decision, constraints, alternatives considered, trade-offs, validation metrics.

**Design doc covers:** data models, API contracts, UI/UX flows, component architecture, service architecture, security design, migration plan.

---

### Phase 4 — Development

This is where code gets written. The flow is: **plan → implement → commit → merge**.

#### `sdlc-sprint` — Plan the Sprint

```bash
/sdlc-launcher:sdlc-sprint MA-2 1
```

What happens:
1. Fetches backlog stories for the epic from Jira
2. Calculates team capacity (dev count × days × 0.7 factor)
3. Selects stories by priority, fitting within capacity
4. Creates sprint in Jira with goal and dates
5. Assigns committed stories to the sprint
6. **Creates Git branches:**
   - `sprint/1-foundation` branched from `main`
   - `story/{KEY}-{slug}` branched from sprint branch (one per dev story)

#### `sdlc-plan` — Create Implementation Plan

```bash
/sdlc-launcher:sdlc-plan MA-3
```

Reads the story, design doc, and codebase. Produces:
- File: `.claude/SDLCs/plans/plan-MA-3.md`
- 3–7 implementation phases with file targets, estimates, validation commands
- Jira sub-tasks created under the story (one per phase)
- Story transitioned to "In Progress"

#### `sdlc-implement` — Write the Code

```bash
/sdlc-launcher:sdlc-implement MA-3
```

Executes the plan phase by phase:
- Reads every existing file before touching it
- Writes clean, minimal code following codebase conventions
- Validates after each phase (typecheck, lint, tests)
- **Independent phases run in parallel via Overstory** (isolated worktree agents)
- Sequential phases run in order in the main agent
- Logs time on Jira sub-tasks
- Transitions sub-tasks to Done as each phase completes

#### `sdlc-commit` — Commit and Open PR

```bash
/sdlc-launcher:sdlc-commit MA-3
```

1. Reads changed files and story details
2. Generates conventional commit message: `feat(MA-3): create financial task with CRUD`
3. Stages relevant files, commits
4. Opens PR: `story/MA-3-create-task` → `sprint/1-foundation` (**not** main)
5. PR body includes: story summary, AC checklist, Jira link, changed files
6. Transitions MA-3 to "In Review" in Jira

#### `sdlc-merge` — Guarded Merge (Story → Sprint)

```bash
/sdlc-launcher:sdlc-merge MA-3
```

**Guard check — Jira status must be Done:**

```
IF MA-3 status == "Done"
  → Opens PR: story/MA-3-create-task → sprint/1-foundation ✓

IF MA-3 status == "In Progress" or anything else
  → BLOCKED ✗
  ✗ MERGE BLOCKED — MA-3 is not Done
  Current status : In Progress
  Required status: Done
  Complete implementation first: /sdlc-implement MA-3
```

The user reviews the PR in GitHub and merges it. The code is now in the sprint branch.

---

### Phase 5 — Testing

| Skill | What it does | Input | Output |
|-------|-------------|-------|--------|
| `sdlc-test` | Generates test plan and test cases | Story key | `.claude/SDLCs/test-reports/test-plan-{KEY}.md` + Jira comment |
| `sdlc-qa` | Runs QA checklist against acceptance criteria | Story key | QA report + Jira status update |
| `sdlc-bug` | Files and triages a bug | Description | Jira Bug ticket + `.claude/SDLCs/` bug record |

```bash
/sdlc-launcher:sdlc-test MA-3
/sdlc-launcher:sdlc-qa MA-3
/sdlc-launcher:sdlc-bug "Amount field accepts negative values"
```

---

### Phase 6 — Deployment

#### `sdlc-merge` — Guarded Merge (Sprint → Main)

When all stories in the sprint are Done:

```bash
/sdlc-launcher:sdlc-merge sprint/1
```

**Guard checks:**
1. Fetches all stories in sprint from Jira
2. Checks every story status == Done
3. Checks no open story PRs remain on the sprint branch

```
If all pass:
  → Opens PR: sprint/1-foundation → main
  → Sprint closed in Jira

If any story is not Done:
  ✗ SPRINT MERGE BLOCKED
  MA-12  Authentication  [In Progress]
  → /sdlc-merge MA-12 first
```

The user merges the sprint PR in GitHub. The sprint code is now on `main`.

#### `sdlc-release` — Tag the Release

```bash
/sdlc-launcher:sdlc-release MA-2 v1.0.0
```

First checks the sprint PR is merged to `main`. Then:
- Creates `.claude/SDLCs/releases/release-v1.0.0.md` with full release notes
- Creates Jira Version (fix version) for all included stories
- Creates GitHub tag `v1.0.0` on `main`

#### `sdlc-deploy` — Deploy to Environment

```bash
/sdlc-launcher:sdlc-deploy v1.0.0 staging
/sdlc-launcher:sdlc-deploy v1.0.0 production
```

Runs deployment checklist, logs deployment in Jira, updates story fix versions.

---

### Phase 7 — Maintenance

| Skill | What it does | Input |
|-------|-------------|-------|
| `sdlc-retro` | Facilitates sprint retrospective, captures action items | Sprint key |
| `sdlc-monitor` | Checks production health, creates incident tickets | Version key |
| `sdlc-hotfix` | Creates emergency hotfix branch, implements, and merges to main + sprint | Bug description |

```bash
/sdlc-launcher:sdlc-retro sprint/1
/sdlc-launcher:sdlc-monitor v1.0.0
/sdlc-launcher:sdlc-hotfix "amount field accepts negative numbers"
```

---

## 5. Git Branching Model

```
main  ──────────────────────────────────────────────► (protected, releases only)
  │
  └─► sprint/1-foundation
        │
        ├─► story/MA-3-create-task
        │     └── PR → sprint/1-foundation  (guard: MA-3 must be Done)
        │
        ├─► story/MA-5-complete-task
        │     └── PR → sprint/1-foundation  (guard: MA-5 must be Done)
        │
        └─► story/MA-12-authentication
              └── PR → sprint/1-foundation  (guard: MA-12 must be Done)
        │
        └── PR → main  (guard: ALL stories must be Done)
              │
              └── tag: v1.0.0

  └─► sprint/2-core-features
        ├─► story/MA-6-recurring-tasks
        └─► story/MA-7-dashboard
```

**Branch naming:**

| Branch | Created by | Pattern | Example |
|--------|-----------|---------|---------|
| Sprint | `sdlc-sprint` | `sprint/{N}-{goal-slug}` | `sprint/1-foundation` |
| Story | `sdlc-sprint` | `story/{KEY}-{title-slug}` | `story/MA-3-create-task` |
| Main | pre-existing | `main` | `main` |

**Branch protection:**
- `main` — protected, no direct push, requires PR
- `sprint/*` — protected, only story PRs accepted
- `story/*` — open, developer pushes freely

**Which stories get a branch:**
- ✅ Dev stories: feature, implementation, bug fix, chore with code output
- ❌ Skipped: UX/design, research/spike, QA-only stories

---

## 6. Jira ↔ Local Sync

### Push direction: Local → Jira
Every skill writes to both local files and Jira automatically. No manual step needed.

### Pull direction: Jira → Local

```bash
/sdlc-launcher:sdlc-pull
```

Use this when you've updated Jira manually (changed status, edited AC, re-pointed stories).

What it does:
1. Asks what you changed (epic / stories / sprint / specific key)
2. Fetches latest from Jira via MCP
3. Diffs against local files
4. Shows you what changed before writing anything
5. Updates local files, preserving local-only sections

```
JIRA PULL REPORT
────────────────────────────────────────
✓ MA-3  status: To Do → In Progress
✓ MA-5  points: 2 → 3
✓ MA-7  acceptance criteria: updated
  MA-9  no changes
✗ MA-6  local file missing → created
────────────────────────────────────────
Apply changes? (yes / no / show-diff)
```

---

## 7. Background Tools: Mulch, Seeds, Overstory

### Mulch — expertise memory across sessions

Mulch stores conventions, patterns, decisions, and failures discovered during your work. It prevents repeating mistakes and re-explaining decisions every session.

| When | Command | What it does |
|------|---------|-------------|
| Session open (auto) | `mulch prime --context` | Loads expertise for changed files |
| Before implementing | `mulch search "topic"` | Checks prior decisions |
| Before finishing | `mulch learn` | Reviews what's worth recording |
| Before finishing | `mulch record <domain> --type decision --description "..."` | Saves the insight |
| Before finishing | `mulch sync` | Commits the knowledge |

All 24 skills include Mulch protocol in their session close checklist.

### Seeds — local issue tracking alongside Jira

Seeds is git-native issue tracking. It runs alongside Jira — Seeds tracks your local work state, Jira tracks the team-visible state.

| When | Command | What it does |
|------|---------|-------------|
| Session open (auto) | `sd prime` | Loads open issues + rules |
| Starting a task | `sd create --title "..."` + `sd update <id> --status in_progress` | Claim work |
| Finishing a task | `sd close <id>` | Mark done |
| Before pushing | `sd sync` | Sync Seeds with git |

### Overstory — parallel agent execution

Overstory spawns isolated Claude agents on separate worktrees for tasks that are fully independent.

Used in:
- **`sdlc-kickoff`**: PRD + Stories + ADRs generated simultaneously (3 parallel agents)
- **`sdlc-implement`**: independent implementation phases run in parallel

```bash
overstory spawn \
  --branch "overstory/kickoff-prd-MA-2" \
  --task "Generate PRD for epic MA-2 ..." \
  --worktree
```

Sequential phases (where Phase 2 depends on Phase 1 output) always run in order in the main agent.

---

## 8. All Skills at a Glance

```
ONE-SHOT PIPELINE
  sdlc-kickoff     → idea → brief → epic → PRD+stories+ADRs (parallel) → backlog → sprint → branches

PHASE 1 — PLANNING
  sdlc-idea        → capture and structure a raw idea
  sdlc-epic        → convert idea to Jira epic
  sdlc-roadmap     → phased product roadmap

PHASE 2 — REQUIREMENTS
  sdlc-prd         → full Product Requirements Document
  sdlc-stories     → user stories with acceptance criteria + Jira tickets
  sdlc-acceptance  → detailed acceptance criteria per story

PHASE 3 — DESIGN
  sdlc-arch        → Architecture Decision Records (ADRs)
  sdlc-design      → full technical design document

PHASE 4 — DEVELOPMENT
  sdlc-sprint      → sprint planning + Jira sprint + Git branches
  sdlc-plan        → step-by-step implementation plan + Jira sub-tasks
  sdlc-implement   → execute plan, write code, validate per phase
  sdlc-commit      → conventional commit + PR to sprint branch
  sdlc-merge       → guarded merge (story→sprint or sprint→main)

PHASE 5 — TESTING
  sdlc-test        → test plan and test cases
  sdlc-qa          → QA checklist against acceptance criteria
  sdlc-bug         → file and triage a bug

PHASE 6 — DEPLOYMENT
  sdlc-release     → release notes + Jira version + GitHub tag
  sdlc-deploy      → deployment checklist + environment update

PHASE 7 — MAINTENANCE
  sdlc-retro       → sprint retrospective + action items
  sdlc-monitor     → production health check + incident tickets
  sdlc-hotfix      → emergency fix branch → implement → merge

UTILITIES
  sdlc-sprint      → sprint planning (also in Phase 4)
  sdlc-status      → project dashboard (all artifacts, Jira state)
  sdlc-sync-jira   → push local artifacts → Jira (one-way)
  sdlc-pull        → pull Jira changes → local artifacts (reverse sync)
```

---

## 9. Environment Variables Required

| Variable | Used by | Description |
|----------|---------|-------------|
| `JIRA_BASE_URL` | jira MCP | e.g. `https://yourcompany.atlassian.net` |
| `JIRA_EMAIL` | jira MCP | Your Atlassian account email |
| `JIRA_API_TOKEN` | jira MCP | Atlassian API token |
| `CONFLUENCE_URL` | jira MCP | e.g. `https://yourcompany.atlassian.net/wiki` |
| `GITHUB_TOKEN` | github MCP | GitHub personal access token (repo + PR scopes) |

---

## 10. Folder Structure

```
sdlc-launcher/                    ← plugin root (load with --plugin-dir)
├── .claude-plugin/
│   └── plugin.json               ← manifest (v2.3.0)
├── .mcp.json                     ← jira + github MCP definitions
├── hooks/
│   └── hooks.json                ← SessionStart: mulch prime + sd prime
├── skills/                       ← 25 skills, each in its own folder
│   ├── sdlc-kickoff/SKILL.md
│   ├── sdlc-idea/SKILL.md
│   ├── sdlc-epic/SKILL.md
│   ├── sdlc-roadmap/SKILL.md
│   ├── sdlc-prd/SKILL.md
│   ├── sdlc-stories/SKILL.md
│   ├── sdlc-acceptance/SKILL.md
│   ├── sdlc-arch/SKILL.md
│   ├── sdlc-design/SKILL.md
│   ├── sdlc-sprint/SKILL.md
│   ├── sdlc-plan/SKILL.md
│   ├── sdlc-implement/SKILL.md
│   ├── sdlc-commit/SKILL.md
│   ├── sdlc-merge/SKILL.md       ← guarded merge (story→sprint, sprint→main)
│   ├── sdlc-test/SKILL.md
│   ├── sdlc-qa/SKILL.md
│   ├── sdlc-bug/SKILL.md
│   ├── sdlc-release/SKILL.md
│   ├── sdlc-deploy/SKILL.md
│   ├── sdlc-retro/SKILL.md
│   ├── sdlc-monitor/SKILL.md
│   ├── sdlc-hotfix/SKILL.md
│   ├── sdlc-status/SKILL.md
│   ├── sdlc-sync-jira/SKILL.md
│   └── sdlc-pull/SKILL.md
└── agents/                       ← 6 specialized agents
    ├── git-manager.md            ← branch lifecycle, merge guards
    ├── jira-connector.md         ← Jira API operations
    ├── tech-architect.md         ← architecture decisions
    ├── requirements-analyst.md   ← PRD + story generation
    ├── qa-engineer.md            ← test plans + QA
    └── release-manager.md        ← release + deployment

.claude/
├── settings/
│   └── jira-config.json          ← project key, board ID, field mappings
└── SDLCs/                        ← all generated artifacts
    ├── briefs/                   ← project brief documents
    ├── ideas/                    ← raw idea files
    ├── epics/                    ← epic + roadmap documents
    ├── prds/                     ← product requirements documents
    ├── stories/                  ← user story files (one per story)
    ├── designs/                  ← ADRs + technical design docs
    ├── plans/                    ← implementation plans (one per story)
    ├── test-reports/             ← test plans and QA reports
    ├── releases/                 ← release notes documents
    └── retros/                   ← retrospective notes
```

---

## Quick Start (TL;DR)

```bash
# 1. Set env vars
export JIRA_BASE_URL="..." JIRA_EMAIL="..." JIRA_API_TOKEN="..."
export CONFLUENCE_URL="..." GITHUB_TOKEN="..."

# 2. Load plugin
claude --plugin-dir ./sdlc-launcher

# 3. Go from idea to backlog in one command
/sdlc-launcher:sdlc-kickoff "your product idea"

# 4. Plan and implement each story
/sdlc-launcher:sdlc-plan MA-3
/sdlc-launcher:sdlc-implement MA-3

# 5. Commit and open PR to sprint branch
/sdlc-launcher:sdlc-commit MA-3

# 6. Merge story to sprint (only if Jira status = Done)
/sdlc-launcher:sdlc-merge MA-3

# 7. When all stories done, merge sprint to main
/sdlc-launcher:sdlc-merge sprint/1

# 8. Tag the release
/sdlc-launcher:sdlc-release MA-2 v1.0.0
```
