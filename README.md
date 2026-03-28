# Claude Skills

A composable skills, agents, hooks, and commands plugin for Claude Code and other coding agents — inspired by [obra/superpowers](https://github.com/obra/superpowers).

## Overview

This repository is a scaffold for building custom workflows for your coding agents. Add skills, agents, hooks, and slash-commands over time and they will be automatically discovered by the plugin system.

## Directory Structure

```
.
├── .claude-plugin/       # Claude Code plugin config
│   ├── plugin.json       # Plugin metadata & entry points
│   └── marketplace.json  # Local marketplace config
├── .cursor-plugin/       # Cursor plugin config
│   └── plugin.json
├── agents/               # Sub-agent prompt files (*.md)
├── commands/             # Slash-command prompt files (*.md)
├── hooks/                # Lifecycle hook scripts
│   ├── hooks.json        # Claude Code hook bindings
│   ├── hooks-cursor.json # Cursor hook bindings
│   ├── run-hook.cmd      # Cross-platform hook runner (Unix + Windows)
│   └── session-start     # SessionStart hook — injects active skills
└── skills/               # Skill prompt files (each in its own subdirectory)
                          # Each skill: skills/<name>/SKILL.md
```

## Adding Skills

1. Create a directory under `skills/`, e.g. `skills/my-skill/`
2. Add a `SKILL.md` file describing the skill
3. The `session-start` hook automatically discovers and injects all `SKILL.md` files at the start of each session

## Adding Agents

Create a Markdown file under `agents/`, e.g. `agents/my-agent.md`.

## Adding Commands

Create a Markdown file under `commands/`, e.g. `commands/my-command.md`.

## Adding Hooks

Edit `hooks/hooks.json` (Claude Code) or `hooks/hooks-cursor.json` (Cursor) to bind new lifecycle events to scripts in the `hooks/` directory.

## Installation

### Claude Code

Register the local marketplace and install:

```bash
/plugin marketplace add ./
/plugin install claude-skills@claude-skills-dev
```

### Cursor

Point Cursor at this directory as a plugin source.

## License

MIT
