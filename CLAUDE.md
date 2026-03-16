# SDLC KIT — Claude Code Configuration

> AI-powered SDLC co-pilot: from raw idea to deployed product, with Jira integration at every step.

## Project Overview

This project is a Claude Code plugin that manages the complete Software Development Lifecycle (SDLC) using structured slash commands, AI-driven artifact generation, and deep Jira integration.

**Two MCPs power all external operations:**
- `jira` (mcp-atlassian) — Jira + Confluence
- `github` (@modelcontextprotocol/server-github) — Git operations

**Spec:** See `SPEC.MD` for the full architecture and command reference.

## Directory Structure

The plugin lives in `sdlc-launcher/` and follows the official Claude Code plugin spec:

```
sdlc-launcher/                  # Plugin root (load with --plugin-dir)
├── .claude-plugin/
│   └── plugin.json             # Plugin manifest (name, version, author…)
├── .mcp.json                   # jira + github MCP definitions
├── hooks/
│   └── hooks.json              # SessionStart: mulch prime + sd prime
├── skills/                     # 25 skills — each in its own folder with SKILL.md
│   ├── sdlc-kickoff/SKILL.md   # one-shot: idea → backlog → sprint → branches
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
│   ├── sdlc-merge/SKILL.md     # guarded merge (story→sprint or sprint→main)
│   ├── sdlc-test/SKILL.md
│   ├── sdlc-bug/SKILL.md
│   ├── sdlc-qa/SKILL.md
│   ├── sdlc-release/SKILL.md
│   ├── sdlc-deploy/SKILL.md
│   ├── sdlc-hotfix/SKILL.md
│   ├── sdlc-monitor/SKILL.md
│   ├── sdlc-retro/SKILL.md
│   ├── sdlc-sprint/SKILL.md
│   ├── sdlc-status/SKILL.md
│   ├── sdlc-sync-jira/SKILL.md
│   └── sdlc-pull/SKILL.md      # pull Jira changes → local artifacts
└── agents/                     # 6 specialized AI agents
    ├── git-manager.md           # branch lifecycle, merge guards
    ├── jira-connector.md        # Jira API operations
    ├── tech-architect.md        # architecture decisions
    ├── requirements-analyst.md  # PRD + story generation
    ├── qa-engineer.md           # test plans + QA
    └── release-manager.md       # release + deployment

.claude/
├── settings/
│   └── jira-config.json        # project key, board ID, field mappings
└── SDLCs/                      # all generated artifacts
    ├── briefs/                  # project brief documents
    ├── ideas/                   # raw idea files
    ├── epics/                   # epic + roadmap documents
    ├── prds/                    # product requirements documents
    ├── stories/                 # user story files (one per story)
    ├── designs/                 # ADRs + technical design docs
    ├── plans/                   # implementation plans (one per story)
    ├── test-reports/            # test plans and QA reports
    ├── releases/                # release notes documents
    └── retros/                  # retrospective notes
```

**Load the plugin locally:**
```bash
claude --plugin-dir ./sdlc-launcher
```

## Key Commands

Commands are namespaced under `/sdlc-launcher:` when loaded as a plugin.

| Phase | Commands |
|-------|----------|
| Kickoff (one-shot) | `/sdlc-launcher:sdlc-kickoff` |
| Planning | `/sdlc-launcher:sdlc-idea`, `/sdlc-launcher:sdlc-epic`, `/sdlc-launcher:sdlc-roadmap` |
| Requirements | `/sdlc-launcher:sdlc-prd`, `/sdlc-launcher:sdlc-stories`, `/sdlc-launcher:sdlc-acceptance` |
| Design | `/sdlc-launcher:sdlc-design`, `/sdlc-launcher:sdlc-arch` |
| Development | `/sdlc-launcher:sdlc-sprint`, `/sdlc-launcher:sdlc-plan`, `/sdlc-launcher:sdlc-implement`, `/sdlc-launcher:sdlc-commit`, `/sdlc-launcher:sdlc-merge` |
| Testing | `/sdlc-launcher:sdlc-test`, `/sdlc-launcher:sdlc-bug`, `/sdlc-launcher:sdlc-qa` |
| Deployment | `/sdlc-launcher:sdlc-release`, `/sdlc-launcher:sdlc-deploy` |
| Maintenance | `/sdlc-launcher:sdlc-retro`, `/sdlc-launcher:sdlc-monitor`, `/sdlc-launcher:sdlc-hotfix` |
| Utils | `/sdlc-launcher:sdlc-status`, `/sdlc-launcher:sdlc-sync-jira`, `/sdlc-launcher:sdlc-sprint`, `/sdlc-launcher:sdlc-pull` |

## Design Principles

1. **Every action produces a Jira artifact** — no SDLC step is undocumented
2. **Commands are composable** — each command runs standalone or in a pipeline
3. **AI-first, human-validated** — Claude generates, humans approve critical actions
4. **Context-aware** — commands read project context, Jira state, and codebase before acting
5. **Traceability** — every commit, PR, and code change links back to a Jira issue

## Setup

1. Set required environment variables:
   ```bash
   export JIRA_BASE_URL="https://yourcompany.atlassian.net"
   export JIRA_EMAIL="you@yourcompany.com"
   export JIRA_API_TOKEN="your-api-token"
   export CONFLUENCE_URL="https://yourcompany.atlassian.net/wiki"
   export GITHUB_TOKEN="your-github-pat"
   ```
2. Configure project settings in `.claude/settings/jira-config.json` (project key, board ID, field mappings)
3. Load the plugin: `claude --plugin-dir ./sdlc-launcher`
4. Go from idea to full backlog in one command:
   ```bash
   /sdlc-launcher:sdlc-kickoff "your product idea"
   ```
   Or run individual phase commands, e.g. `/sdlc-launcher:sdlc-idea "your idea here"`

<!-- mulch:start -->
## Project Expertise (Mulch)
<!-- mulch-onboard-v:1 -->

This project uses [Mulch](https://github.com/jayminwest/mulch) for structured expertise management.

**At the start of every session**, run:
```bash
mulch prime
```

This injects project-specific conventions, patterns, decisions, and other learnings into your context.
Use `mulch prime --files src/foo.ts` to load only records relevant to specific files.

**Before completing your task**, review your work for insights worth preserving — conventions discovered,
patterns applied, failures encountered, or decisions made — and record them:
```bash
mulch record <domain> --type <convention|pattern|failure|decision|reference|guide> --description "..."
```

Link evidence when available: `--evidence-commit <sha>`, `--evidence-bead <id>`

Run `mulch status` to check domain health and entry counts.
Run `mulch --help` for full usage.
Mulch write commands use file locking and atomic writes — multiple agents can safely record to the same domain concurrently.

### Before You Finish

1. Discover what to record:
   ```bash
   mulch learn
   ```
2. Store insights from this work session:
   ```bash
   mulch record <domain> --type <convention|pattern|failure|decision|reference|guide> --description "..."
   ```
3. Validate and commit:
   ```bash
   mulch sync
   ```
<!-- mulch:end -->

<!-- seeds:start -->
## Issue Tracking (Seeds)
<!-- seeds-onboard-v:1 -->

This project uses [Seeds](https://github.com/jayminwest/seeds) for git-native issue tracking.

**At the start of every session**, run:
```
sd prime
```

This injects session context: rules, command reference, and workflows.

**Quick reference:**
- `sd ready` — Find unblocked work
- `sd create --title "..." --type task --priority 2` — Create issue
- `sd update <id> --status in_progress` — Claim work
- `sd close <id>` — Complete work
- `sd dep add <id> <depends-on>` — Add dependency between issues
- `sd sync` — Sync with git (run before pushing)

### Before You Finish
1. Close completed issues: `sd close <id>`
2. File issues for remaining work: `sd create --title "..."`
3. Sync and push: `sd sync && git push`
<!-- seeds:end -->
