# SDLC KIT — Claude Code Configuration

> AI-powered SDLC co-pilot: from raw idea to deployed product, with Jira integration at every step.

## Project Overview

This project is a Claude Code plugin that manages the complete Software Development Lifecycle (SDLC) using structured slash commands, AI-driven artifact generation, and deep Jira integration.

**Spec:** See `SPEC.MD` for the full architecture and command reference.

## Directory Structure

The plugin lives in `sdlc-launcher/` and follows the official Claude Code plugin spec:

```
sdlc-launcher/                  # Plugin root
├── .claude-plugin/
│   └── plugin.json             # Plugin manifest (name, version, author…)
├── skills/                     # Skills — each in its own folder with SKILL.md
│   ├── sdlc-idea/SKILL.md
│   ├── sdlc-epic/SKILL.md
│   ├── sdlc-roadmap/SKILL.md
│   ├── sdlc-prd/SKILL.md
│   ├── sdlc-stories/SKILL.md
│   ├── sdlc-acceptance/SKILL.md
│   ├── sdlc-arch/SKILL.md
│   ├── sdlc-design/SKILL.md
│   ├── sdlc-plan/SKILL.md
│   ├── sdlc-implement/SKILL.md
│   ├── sdlc-commit/SKILL.md
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
│   └── sdlc-sync-jira/SKILL.md
└── agents/                     # Specialized AI agents (jira, architect, QA, etc.)

.claude/
├── SDLCs/                      # Generated artifacts (ideas, PRDs, epics, plans…)
└── settings/                   # Jira config and plugin settings
```

**Load the plugin locally:**
```bash
claude --plugin-dir ./sdlc-launcher
```

## Key Commands

Commands are namespaced under `/sdlc-launcher:` when loaded as a plugin.

| Phase | Commands |
|-------|----------|
| Planning | `/sdlc-launcher:sdlc-idea`, `/sdlc-launcher:sdlc-epic`, `/sdlc-launcher:sdlc-roadmap` |
| Requirements | `/sdlc-launcher:sdlc-prd`, `/sdlc-launcher:sdlc-stories`, `/sdlc-launcher:sdlc-acceptance` |
| Design | `/sdlc-launcher:sdlc-design`, `/sdlc-launcher:sdlc-arch` |
| Development | `/sdlc-launcher:sdlc-plan`, `/sdlc-launcher:sdlc-implement`, `/sdlc-launcher:sdlc-commit` |
| Testing | `/sdlc-launcher:sdlc-test`, `/sdlc-launcher:sdlc-bug`, `/sdlc-launcher:sdlc-qa` |
| Deployment | `/sdlc-launcher:sdlc-release`, `/sdlc-launcher:sdlc-deploy` |
| Maintenance | `/sdlc-launcher:sdlc-retro`, `/sdlc-launcher:sdlc-monitor`, `/sdlc-launcher:sdlc-hotfix` |
| Utils | `/sdlc-launcher:sdlc-status`, `/sdlc-launcher:sdlc-sync-jira`, `/sdlc-launcher:sdlc-sprint` |

## Design Principles

1. **Every action produces a Jira artifact** — no SDLC step is undocumented
2. **Commands are composable** — each command runs standalone or in a pipeline
3. **AI-first, human-validated** — Claude generates, humans approve critical actions
4. **Context-aware** — commands read project context, Jira state, and codebase before acting
5. **Traceability** — every commit, PR, and code change links back to a Jira issue

## Setup

1. Configure Jira credentials in `.claude/settings/jira-config.json`
2. Load the plugin: `claude --plugin-dir ./sdlc-launcher`
3. Run any command, e.g. `/sdlc-launcher:sdlc-idea "your idea here"`

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
