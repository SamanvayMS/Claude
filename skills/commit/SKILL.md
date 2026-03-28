# Commit Skill

Activate this skill when the user asks you to **commit**, **make a commit**, or **run the commit workflow**.

---

## Overview

The commit workflow follows these steps in order:

1. Read project documentation (Markdown files)
2. Detect and run tests — stop with suggestions on failure
3. Review all uncommitted changes
4. Determine the next version number
5. Update or create `CHANGELOG.md`
6. Craft a commit message
7. Stage all changes and commit

---

## Step 1 — Read Project Documentation

Before doing anything else, find and read every Markdown file in the project to understand the intent, architecture, and conventions:

```bash
find . -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*"
```

Read each file. Pay particular attention to:
- `README.md` — project purpose and structure
- Any `ARCHITECTURE.md`, `CONTRIBUTING.md`, or `DESIGN.md` — coding conventions
- `CHANGELOG.md` — existing version history (if present)
- Any `docs/` subdirectory

---

## Step 2 — Detect and Run Tests

### 2a. Detect if tests exist

Check for any of the following indicators:

| Indicator | How to check |
|-----------|--------------|
| `package.json` has a `test` script | `jq -r '.scripts.test // empty' package.json` |
| `pytest` / `unittest` test files | `find . -name "test_*.py" -o -name "*_test.py"` (excluding `node_modules/` and `.git/`) |
| Jest / Mocha / Vitest test files | `find . -name "*.test.*" -o -name "*.spec.*"` (excluding `node_modules/` and `.git/`) |
| Go test files | `find . -name "*_test.go"` |
| Makefile with `test` target | `grep -q "^test:" Makefile 2>/dev/null` |
| Any `tests/` or `test/` directory | `[ -d tests ] || [ -d test ]` |

If **no tests are found**, skip to Step 3 and note "No tests found" in the commit message footer.

### 2b. Run the tests

Run the appropriate command based on what was detected:

- **npm/yarn/pnpm project**: `npm test` (or `yarn test` / `pnpm test`)
- **Python (pytest)**: `pytest`
- **Python (unittest)**: `python -m unittest discover`
- **Go**: `go test ./...`
- **Makefile test target**: `make test`
- **Multiple ecosystems**: run all applicable commands

### 2c. Handle test failures — STOP and give suggestions

**If any test fails, you MUST stop the commit workflow immediately.**

Do NOT proceed to Step 3. Instead:

1. Output a clear `❌ COMMIT ABORTED — Tests failed` header.
2. Show the full test failure output.
3. Provide actionable suggestions for each failure:
   - Identify the failing test name and the assertion that failed.
   - Suggest the most likely fix (e.g., update the implementation to match the expectation, or update the test if the requirement changed).
   - If the error is a configuration/environment issue (missing dependency, wrong Node version, etc.), explain how to resolve it.
4. Ask the user: *"Would you like me to attempt to fix these failures before committing?"*

**Only continue to Step 3 after ALL tests pass.**

---

## Step 3 — Review All Uncommitted Changes

Run:

```bash
git diff HEAD
git status --short
```

Read and understand every change:
- What files were added, modified, or deleted?
- What is the nature of the changes (new feature, bug fix, refactor, docs, config)?
- Are there any obvious mistakes, leftover debug code, or accidental changes?

If you notice obvious mistakes (e.g., `console.log` debug statements, commented-out code blocks, merge conflict markers), point them out to the user and ask if they should be cleaned up before committing.

---

## Step 4 — Determine the Next Version Number

The version follows the format `MAJOR.MINOR.PATCH` (e.g., `0.1.3`).

**Rules:**
- **PATCH** (3rd number) — increments with every commit on any branch.
- **MINOR** (2nd number) — increments when a branch is merged into `main`. When MINOR increments, PATCH resets to `0`.
- **MAJOR** (1st number) — only incremented manually by the developer; never auto-incremented by this skill.

### Finding the current version

Check in this priority order:

1. `CHANGELOG.md` — look for the most recent version header (`## [X.Y.Z]` or `## X.Y.Z`).
2. `package.json` — read the `version` field.
3. Any `VERSION` or `version.txt` file.
4. If no version is found anywhere, start at `0.0.0`.

### Calculating the next version

Detect the current branch:

```bash
git rev-parse --abbrev-ref HEAD
```

- If the branch **is `main`** (or `master`): increment MINOR, reset PATCH to `0`.
- Otherwise (feature branch, fix branch, etc.): increment PATCH only.

**Example:**
- Current version `0.1.4` on a feature branch → next version `0.1.5`
- Current version `0.1.5` on `main` (after merge) → next version `0.2.0`

### Update version in source files

After calculating the new version, update it in every file that declares the version:

- `package.json` → `"version": "X.Y.Z"`
- `.claude-plugin/plugin.json` → `"version": "X.Y.Z"` (if present)
- `.cursor-plugin/plugin.json` → `"version": "X.Y.Z"` (if present)
- `.claude-plugin/marketplace.json` → `"version": "X.Y.Z"` (if present)
- `VERSION` or `version.txt` → replace the single line
- Any other version declarations you discovered in Step 1

---

## Step 5 — Update or Create CHANGELOG.md

### If CHANGELOG.md does not exist — create it

Create `CHANGELOG.md` at the repository root with:

1. A **header** describing the repository (synthesised from the README and your understanding of the codebase — one short paragraph).
2. The standard Keep-a-Changelog preamble.
3. The first version entry (see format below).

Template for a new CHANGELOG:

```markdown
# Changelog

<!-- One-paragraph description of the repository synthesised from README.md -->
<repository description here>

All notable changes to this project will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [X.Y.Z] — YYYY-MM-DD

### <Category>

- <change description>
```

### If CHANGELOG.md already exists — prepend a new entry

Add a new section at the top of the existing version history (below the header/preamble) in this format:

```markdown
## [X.Y.Z] — YYYY-MM-DD

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Removed
- ...
```

Only include categories that are relevant to this commit. Write entries in past-tense to match the Keep a Changelog convention (e.g., "Added commit skill" not "Add commit skill").

---

## Step 6 — Craft the Commit Message

Write a commit message following the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<optional scope>): <short summary>  [v X.Y.Z]

<body — what changed and why, wrapped at 72 chars>

<footer>
- Tests: <passed N/N | no tests found>
- Version: X.Y.Z
- Changelog: updated
```

**Types:** `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`, `perf`, `ci`, `build`

Keep the subject line to 72 characters or fewer. The `[v X.Y.Z]` tag at the end of the subject line makes the version immediately visible in `git log --oneline`.

**Example:**

```
feat(skills): add commit workflow skill  [v 0.1.0]

Introduce the 'commit' skill that automates the full pre-commit
workflow: reads docs, runs tests, reviews changes, bumps version,
updates CHANGELOG, and creates a structured commit message.

- Tests: no tests found
- Version: 0.1.0
- Changelog: created
```

---

## Step 7 — Stage and Commit

```bash
git add -A
git commit -m "<commit message from Step 6>"
```

After committing, confirm success by showing:

```bash
git --no-pager log --oneline -3
```

And report:
> ✅ **Committed successfully as v X.Y.Z**
> Commit: `<short SHA> <subject>`

---

## Error Handling

| Situation | Action |
|-----------|--------|
| No changes to commit (`git status` is clean) | Report "Nothing to commit — working tree clean" and stop |
| Tests fail | Stop immediately; provide per-failure suggestions (see Step 2c) |
| Version file not found | Start from `0.0.0` and note this in the commit footer |
| Partial staging issues | Show `git status` and ask user to resolve before continuing |
| Any command exits non-zero unexpectedly | Show the error, explain the likely cause, and ask the user how to proceed |
