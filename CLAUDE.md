# CLAUDE.md

## Project Overview

Superpowers is a composable skills library for AI coding agents (Claude Code, Cursor, Codex, OpenCode, Gemini CLI). It provides a structured software development workflow through automatically-triggered skills that enforce TDD, systematic debugging, brainstorming-before-coding, and subagent-driven development.

**Author:** Jesse Vincent (jesse@fsck.com)
**License:** MIT
**Version:** 5.0.5
**Repository:** https://github.com/obra/superpowers

## Repository Structure

```
superpowers/
├── skills/                    # Core skills library (main content)
│   ├── <skill-name>/
│   │   ├── SKILL.md           # Main skill definition (required)
│   │   └── *.md / *.ts / *.sh # Supporting references/tools
│   └── ...
├── commands/                  # Deprecated slash commands (point to skills)
├── agents/                    # Agent prompt definitions (e.g., code-reviewer.md)
├── hooks/                     # Platform hooks (session-start, etc.)
│   ├── hooks.json             # Hook configuration
│   ├── session-start          # SessionStart hook script (bash)
│   └── run-hook.cmd           # Windows hook runner
├── tests/                     # Test suites
│   ├── claude-code/           # Claude Code integration tests
│   ├── skill-triggering/      # Skill trigger verification tests
│   ├── subagent-driven-dev/   # SDD workflow tests
│   ├── brainstorm-server/     # Brainstorm server tests
│   ├── explicit-skill-requests/ # Explicit invocation tests
│   └── opencode/              # OpenCode platform tests
├── docs/                      # Documentation
│   ├── plans/                 # Historical design/implementation plans
│   ├── superpowers/           # Specs and plans for the project itself
│   └── testing.md             # Testing guide
├── .claude-plugin/            # Claude Code plugin metadata
│   ├── plugin.json            # Plugin manifest
│   └── marketplace.json       # Marketplace listing
├── .codex/                    # Codex platform support
├── .opencode/                 # OpenCode platform support
├── .cursor-plugin/            # Cursor platform support
├── package.json               # ESM module, version info
├── GEMINI.md                  # Gemini CLI entry point
├── CHANGELOG.md               # Release changelog
└── RELEASE-NOTES.md           # Detailed release notes
```

## Skills Architecture

Each skill lives in `skills/<skill-name>/` with a required `SKILL.md` file.

### Skill file format

```markdown
---
name: skill-name-with-hyphens
description: Use when [triggering conditions] - third person, max 1024 chars
---

# Skill Name

## Overview
...
```

**Frontmatter rules:**
- Only `name` and `description` fields are supported
- `name`: letters, numbers, hyphens only (no special chars)
- `description`: Must start with "Use when..." - describe triggers only, NOT workflow
- Max 1024 characters total frontmatter

### Current skills (14 total)

| Skill | Purpose |
|-------|---------|
| `brainstorming` | Socratic design refinement before coding |
| `writing-plans` | Break specs into detailed implementation tasks |
| `executing-plans` | Batch execution with human checkpoints |
| `subagent-driven-development` | Fast iteration via subagents with two-stage review |
| `dispatching-parallel-agents` | Concurrent independent subagent workflows |
| `test-driven-development` | RED-GREEN-REFACTOR cycle enforcement |
| `systematic-debugging` | 4-phase root cause process |
| `verification-before-completion` | Evidence-based completion verification |
| `requesting-code-review` | Pre-merge review checklist |
| `receiving-code-review` | Processing review feedback with rigor |
| `using-git-worktrees` | Isolated development branches |
| `finishing-a-development-branch` | Merge/PR decision workflow |
| `using-superpowers` | Meta skill - how to discover and use skills |
| `writing-skills` | Creating and testing new skills (TDD for docs) |

### Core workflow sequence

1. **brainstorming** - Explore requirements before coding
2. **using-git-worktrees** - Create isolated workspace
3. **writing-plans** - Break work into tasks (2-5 min each)
4. **subagent-driven-development** or **executing-plans** - Execute the plan
5. **test-driven-development** - RED-GREEN-REFACTOR per task
6. **requesting-code-review** - Review between tasks
7. **finishing-a-development-branch** - Integrate completed work

## Development Conventions

### Philosophy

- **Test-Driven Development** - Write tests first, always
- **YAGNI** - You Aren't Gonna Need It
- **DRY** - Don't Repeat Yourself
- **Evidence over claims** - Verify before declaring success
- **Systematic over ad-hoc** - Process over guessing

### Skill creation follows TDD

Creating or editing skills requires the RED-GREEN-REFACTOR cycle:
1. **RED**: Run pressure scenarios WITHOUT the skill, document baseline failures
2. **GREEN**: Write minimal skill addressing those specific failures
3. **REFACTOR**: Close loopholes, add rationalization counters, re-test

### CSO (Claude Search Optimization)

Descriptions must help future Claude instances find the right skill:
- Start with "Use when..." (triggering conditions only)
- Never summarize workflow in description (causes shortcutting)
- Include searchable keywords: error messages, symptoms, tool names
- Use active voice, verb-first naming (e.g., `creating-skills` not `skill-creation`)

### Token efficiency

- Getting-started workflows: <150 words
- Frequently-loaded skills: <200 words
- Other skills: <500 words
- Move heavy reference to separate files
- Use cross-references instead of repeating content

## Hook System

The `hooks/session-start` script runs on SessionStart and injects the `using-superpowers` skill content into the conversation context. It supports Claude Code, Cursor, and generic platforms via different JSON output formats.

Configuration is in `hooks/hooks.json`.

## Testing

### Running tests

```bash
# Unit tests (fast)
cd tests/claude-code && ./run-skill-tests.sh

# Integration tests (10-30 min, requires Claude CLI)
cd tests/claude-code && ./run-skill-tests.sh --integration

# Specific test
./run-skill-tests.sh --test test-subagent-driven-development.sh

# Verbose output
./run-skill-tests.sh --verbose
```

### Requirements

- Claude Code CLI must be installed (`claude` command)
- Run from the superpowers directory (skills only load from plugin root)
- For integration tests: `"superpowers@superpowers-dev": true` in `~/.claude/settings.json`

### Skill triggering tests

```bash
cd tests/skill-triggering && ./run-all.sh
```

### Token analysis

```bash
python3 tests/claude-code/analyze-token-usage.py <session-file.jsonl>
```

## Multi-Platform Support

The project supports multiple AI coding platforms:
- **Claude Code**: Primary platform, uses `.claude-plugin/` manifest and hooks
- **Cursor**: Uses `.cursor-plugin/` and `hooks-cursor.json`
- **Codex**: Manual setup via `.codex/INSTALL.md`
- **OpenCode**: Manual setup via `.opencode/INSTALL.md`, has JS plugin at `.opencode/plugins/superpowers.js`
- **Gemini CLI**: Uses `GEMINI.md` and `gemini-extension.json`

## Build and Package

- `package.json` declares `"type": "module"` (ESM)
- No build step - skills are plain Markdown files
- No runtime dependencies
- Version is tracked in both `package.json` (5.0.4) and `.claude-plugin/plugin.json` (5.0.5)

## Git Conventions

- `main` branch for releases
- `dev` branch for development (merged to main for releases)
- Feature branches for individual changes
- Commit messages should be descriptive
- Use git worktrees for parallel development (the `using-git-worktrees` skill documents this)

## Key Files for Contributors

- `skills/writing-skills/SKILL.md` - Complete guide for creating new skills
- `skills/writing-skills/testing-skills-with-subagents.md` - Testing methodology
- `skills/writing-skills/anthropic-best-practices.md` - Official skill authoring guidance
- `docs/testing.md` - Integration testing guide
- `CHANGELOG.md` - Release history
