# Claude Plugins (Local Marketplace)

Custom Claude Code plugins for Flash Financials development workflows.

## Plugins

| Plugin | Description |
|--------|-------------|
| **flash** | Full dev workflow — services, testing, reports, deploy, production ops, infrastructure |
| **skill-authoring** | Guide for creating Claude Code skills and plugins |
| **git-workflow** | Git worktree commands for feature branch isolation |

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/flash-financials/claude-plugins.git ~/Desktop/claude-plugins
```

### 2. Register as a local marketplace

```bash
claude plugins marketplace add ~/Desktop/claude-plugins
```

This registers the repo as a directory-based marketplace named `local` and creates symlinks in `~/.claude/plugins/marketplaces/local/`.

### 3. Install plugins

```bash
claude plugins install flash@local
claude plugins install git-workflow@local
claude plugins install skill-authoring@local
```

Each install copies the plugin into `~/.claude/plugins/cache/local/<name>/<version>/`.

### 4. Enable plugins

```bash
claude plugins enable flash@local
claude plugins enable git-workflow@local
claude plugins enable skill-authoring@local
```

Or enable them interactively via `/plugins` in Claude Code.

### 5. Restart Claude Code

Restart your Claude Code session to pick up the new plugins.

## Updating After Changes

After pulling new changes or editing plugin files locally:

```bash
# Reinstall to refresh the cache
claude plugins install flash@local --force

# Or manually sync the cache
cp -r ~/Desktop/claude-plugins/flash/* ~/.claude/plugins/cache/local/flash/1.0.0/
```

Then restart Claude Code.

## Flash Plugin — Agents

The flash plugin includes 11 specialized agents:

| Agent | Model | Purpose |
|-------|-------|---------|
| `flash-dev` | opus | Full dev workflow — services, testing, reports, build, deploy |
| `flash-prod` | opus | Production incident response, health monitoring, log analysis |
| `flash-infra` | opus | Terraform infrastructure — plan, apply, state management |
| `code-reviewer` | opus | Code review — quality, security, best practices |
| `security-auditor` | opus | Security audits — auth flows, vulnerabilities, compliance |
| `debugger` | opus | Bug diagnosis — root cause analysis, error tracing |
| `golang-pro` | sonnet | Go development — idiomatic patterns, concurrency, testing |
| `react-specialist` | sonnet | React frontend — components, state, Vite/TypeScript |
| `postgres-pro` | sonnet | PostgreSQL — query optimization, tuning, schema |
| `terraform-engineer` | sonnet | Terraform code review and IaC patterns |
| `performance-engineer` | sonnet | Performance profiling, bottleneck analysis, optimization |

Agents are auto-discovered from `flash/agents/*.md`. No plugin.json changes needed to add new agents.

## Flash Plugin — Slash Commands

Run `/flash` in Claude Code to see all available commands, or invoke specific ones:

| Command | Description |
|---------|-------------|
| `/dev` | Start local dev environment |
| `/dev-stop` | Stop all local services |
| `/status` | Check service status |
| `/test` | Run tests |
| `/build` | Build frontend + backend |
| `/deploy` | Deploy to Cloud Run |
| `/generate` | Generate financial reports |
| `/infra` | Terraform operations |
| `/secrets-get` | Pull secrets from Secret Manager |
| `/prod-status` | Check production health |
| `/prod-logs` | View production logs |
| `/rollback` | Rollback production deployment |

See `flash/commands/` for the full list.

## Directory Structure

```
claude-plugins/
├── marketplace.json          # Plugin registry (lists available plugins)
├── flash/                    # Flash Financials plugin
│   ├── .claude-plugin/
│   │   └── plugin.json       # Plugin metadata
│   ├── agents/               # Subagents (auto-discovered)
│   ├── commands/             # Slash commands
│   └── skills/               # Skills (auto-triggered)
├── git-workflow/             # Git worktree commands
│   ├── .claude-plugin/
│   └── commands/
└── skill-authoring/          # Skill authoring guide
    ├── .claude-plugin/
    └── skills/
```
