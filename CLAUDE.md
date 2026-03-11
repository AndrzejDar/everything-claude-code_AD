# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** - a collection of production-ready agents, skills, hooks, commands, rules, and MCP configurations. The project provides battle-tested workflows for software development using Claude Code.

## Running Tests

```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## Architecture

The project is organized into several core components:

- **agents/** - Specialized subagents for delegation (planner, code-reviewer, tdd-guide, etc.)
- **skills/** - Workflow definitions and domain knowledge (coding standards, patterns, testing)
- **commands/** - Slash commands invoked by users (/tdd, /plan, /e2e, etc.)
- **hooks/** - Trigger-based automations (session persistence, pre/post-tool hooks)
- **rules/** - Always-follow guidelines (security, coding style, testing requirements)
- **mcp-configs/** - MCP server configurations for external integrations
- **scripts/** - Cross-platform Node.js utilities for hooks and setup
- **tests/** - Test suite for scripts and utilities

## Key Commands

- `/tdd` - Test-driven development workflow
- `/plan` - Implementation planning
- `/e2e` - Generate and run E2E tests
- `/code-review` - Quality review
- `/build-fix` - Fix build errors
- `/learn` - Extract patterns from sessions
- `/skill-create` - Generate skills from git history

## Plugin Update After Changes (IMPORTANT)

The plugin system reads from a **cached copy** at `~/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.8.0/`, NOT from the local repo working tree. After adding or modifying agents, skills, commands, hooks, or rules:

1. **Update the cache** — copy changed files into the cache directory:
   ```bash
   # Example: copy a new or modified skill
   cp -r skills/my-new-skill ~/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.8.0/skills/

   # Example: copy a new or modified agent
   cp agents/my-agent.md ~/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.8.0/agents/
   ```
2. **Update the commit SHA** in `~/.claude/plugins/installed_plugins.json` to match the new commit:
   ```bash
   git rev-parse HEAD  # get the SHA, then edit installed_plugins.json
   ```
3. **Reload** — run `/reload-plugins` in Claude Code

Without steps 1-2, new or modified content will NOT appear even after `/reload-plugins`.

## Development Notes

- Package manager detection: npm, pnpm, yarn, bun (configurable via `CLAUDE_PACKAGE_MANAGER` env var or project config)
- Cross-platform: Windows, macOS, Linux support via Node.js scripts
- Agent format: Markdown with YAML frontmatter (name, description, tools, model)
- Skill format: Markdown with clear sections for when to use, how it works, examples
- Hook format: JSON with matcher conditions and command/notification hooks

## Contributing

Follow the formats in CONTRIBUTING.md:
- Agents: Markdown with frontmatter (name, description, tools, model)
- Skills: Clear sections (When to Use, How It Works, Examples)
- Commands: Markdown with description frontmatter
- Hooks: JSON with matcher and hooks array

File naming: lowercase with hyphens (e.g., `python-reviewer.md`, `tdd-workflow.md`)
