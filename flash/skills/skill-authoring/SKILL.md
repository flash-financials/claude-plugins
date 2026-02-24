---
name: skill-authoring
description: Guide for creating Claude Code skills and plugins. Use when the user asks how to make a skill, create a command, build a plugin, or extend Claude Code.
---

Help the user create a Claude Code skill or plugin. Follow the patterns below.

## Skills overview

A skill is a `SKILL.md` file with YAML frontmatter + markdown instructions. Skills can auto-trigger based on conversation context or be invoked manually via `/skill-name`.

### Where skills live

| Location | Path | Scope |
|----------|------|-------|
| Personal | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | Where plugin is enabled |

### Minimal example

```yaml
---
name: my-skill
description: What this does and when to use it
---

Instructions for Claude when this skill is invoked.
```

### Frontmatter fields

| Field | Description |
|-------|-------------|
| `name` | Slash command name (lowercase, hyphens). Defaults to directory name |
| `description` | When to use it. Claude reads this to decide auto-invocation |
| `argument-hint` | Shown in autocomplete, e.g. `[filename]` |
| `disable-model-invocation` | `true` = only user can invoke via `/name` |
| `user-invocable` | `false` = hidden from `/` menu, Claude-only |
| `allowed-tools` | Tools Claude can use without asking, e.g. `Bash(gcloud *), Read` |
| `context` | `fork` = run in isolated subagent |
| `agent` | Subagent type when `context: fork`, e.g. `Explore` |
| `model` | Model override when skill is active |

### Dynamic context injection

Use exclamation mark + backtick-wrapped command to run shell commands before the skill content reaches Claude. The syntax is: `!` immediately followed by a backtick-wrapped shell command.

Example: a pr-review skill would include lines like:
- `PR diff:` followed by bang-backtick `gh pr diff` bang-backtick
- `Changed files:` followed by bang-backtick `gh pr diff --name-only` bang-backtick

These get executed at load time and their output is injected into the skill content before Claude sees it.

### Arguments

Use `$ARGUMENTS` for all args, `$ARGUMENTS[0]` or `$0` for positional:

```yaml
---
name: fix-issue
description: Fix a GitHub issue
argument-hint: "[issue-number]"
---

Fix GitHub issue #$0. Read the issue, implement the fix, write tests.
```

### Supporting files

Keep `SKILL.md` focused. Add reference material as separate files:

```
my-skill/
├── SKILL.md           # Main instructions (required)
├── template.md        # Template for Claude to fill in
├── examples/
│   └── sample.md      # Example output
└── scripts/
    └── helper.sh      # Script Claude can execute
```

Reference them from SKILL.md: `For examples, see [examples/sample.md](examples/sample.md)`

## Plugins

Plugins package skills (plus agents, hooks, MCP servers) for sharing. Skills get namespaced: `/plugin-name:skill-name`.

### Plugin structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # Manifest (only this goes in .claude-plugin/)
├── commands/             # Slash commands as .md files
│   └── deploy.md
├── skills/               # Auto-triggerable skills
│   └── code-review/
│       └── SKILL.md
├── agents/               # Custom subagents
├── hooks/                # Event handlers
│   └── hooks.json
└── .mcp.json             # MCP server configs
```

### plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "author": { "name": "Your Name" },
  "skills": "./skills/",
  "commands": "./commands/"
}
```

### Testing locally

```bash
claude --plugin-dir ./my-plugin
```

### Publishing via marketplace

1. Create `marketplace-dir/.claude-plugin/marketplace.json`:

```json
{
  "name": "my-marketplace",
  "owner": { "name": "Your Name" },
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./my-plugin",
      "description": "What it does",
      "version": "1.0.0"
    }
  ]
}
```

2. Add and install:

```
/plugin marketplace add ./marketplace-dir
/plugin install my-plugin@my-marketplace
```

## Common patterns

**Task command** (user-invoked only, has side effects):
```yaml
---
name: deploy
description: Deploy to production
disable-model-invocation: true
allowed-tools: Bash(*)
---
```

**Reference skill** (auto-triggers, adds knowledge):
```yaml
---
name: api-conventions
description: API design patterns for this codebase
---
```

**Background knowledge** (Claude-only, not in `/` menu):
```yaml
---
name: legacy-context
description: How the old auth system works
user-invocable: false
---
```

**Forked research** (runs in isolated subagent):
```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---
```

## Workflow

When helping the user create a skill:

1. Ask what the skill should do and whether it's user-invoked, auto-triggered, or both
2. Determine scope: personal (`~/.claude/skills/`), project (`.claude/skills/`), or plugin
3. Create the `SKILL.md` with appropriate frontmatter
4. If it needs shell preprocessing, use the bang-backtick dynamic context syntax
5. If it's a plugin, create the directory structure with `.claude-plugin/plugin.json`
6. Test with `claude --plugin-dir` for plugins, or just restart for personal/project skills
