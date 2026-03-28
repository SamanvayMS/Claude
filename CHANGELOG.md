# Changelog

A composable skills, agents, hooks, and commands plugin for Claude Code and other coding agents. Inspired by [obra/superpowers](https://github.com/obra/superpowers), this scaffold lets you extend your AI coding assistant with reusable prompt-based skills, sub-agent definitions, slash-commands, and lifecycle hooks that are automatically discovered and injected at the start of every session.

All notable changes to this project will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.0.3] — 2026-03-28

### Added

- `skills/commit/SKILL.md`: new **commit** skill that automates the full pre-commit workflow — reads project documentation, detects and runs tests (aborting with suggestions on failure), reviews uncommitted changes, auto-increments the version (patch on every commit, minor on merge to main), updates or creates `CHANGELOG.md`, crafts a Conventional Commits message, and stages and commits all changes.
- `CHANGELOG.md`: initial changelog with full repository description.

---

## [0.0.2] — 2026-03-28

### Added

- Superpowers-style scaffold: plugin configurations for Claude Code (`.claude-plugin/`) and Cursor (`.cursor-plugin/`), lifecycle hooks (`hooks/session-start`), and empty placeholder directories for skills, agents, and commands.
- Cross-platform hook runner (`hooks/run-hook.cmd`) supporting both Unix and Windows environments.

---

## [0.0.1] — 2026-03-28

### Added

- Initial repository setup with `README.md`, `LICENSE` (MIT), `package.json`, and `.gitignore`.
