# Plugin Manifest Schema Notes

This document captures **undocumented but enforced constraints** of the Claude Code plugin manifest validator.

These rules are based on real installation failures, validator behavior, and comparison with known working plugins.
They exist to prevent silent breakage and repeated regressions.

If you edit `.claude-plugin/plugin.json`, read this first.

---

## Summary (Read This First)

Claude Code v2.1+ **auto-discovers plugin components by convention** from standard directory names. You do NOT need to declare `agents`, `commands`, `skills`, or `hooks` in `plugin.json`.

The `plugin.json` should contain **only metadata** (name, version, description, author, etc.). Component arrays are unnecessary and may interfere with auto-discovery or cause path resolution failures on Windows.

---

## Convention-Based Auto-Discovery

Claude Code automatically finds plugin components from these conventional paths relative to the plugin root:

| Component | Convention | Format |
|-----------|-----------|--------|
| Agents | `agents/*.md` | Markdown with YAML frontmatter |
| Commands | `commands/*.md` | Markdown with description frontmatter |
| Skills | `skills/*/SKILL.md` | Markdown with sections |
| Hooks | `hooks/hooks.json` | JSON with matcher and hooks array |
| Rules | `rules/**/*.md` | Markdown guidelines |

**No declaration in `plugin.json` is needed.** Place files in the correct directories and Claude Code will find them.

### Proof: agent-coder Plugin

The `agent-coder` plugin works perfectly with a minimal `plugin.json` containing only `name`, `version`, and `description` — zero component arrays. Claude Code auto-discovers all 7 agents, 5 commands, and 5+ skills from conventional directory names.

---

## Required Fields

### `version` (MANDATORY)

The `version` field is required by the validator even if omitted from some examples.

If missing, installation may fail during marketplace install or CLI validation.

---

## DO NOT Declare Component Arrays

> **CRITICAL:** Do NOT add `agents`, `commands`, `skills`, or `hooks` arrays to `plugin.json`.

### Why This Matters

Claude Code v2.1+ automatically loads components from conventional directory paths. Declaring them explicitly in `plugin.json`:

- Is unnecessary — auto-discovery handles everything
- May interfere with the auto-discovery mechanism
- Can cause path resolution failures, especially on Windows
- For `hooks` specifically, causes a duplicate detection error

### The `hooks` Flip-Flop History

The hooks field has caused repeated fix/revert cycles in this repo:

| Commit | Action | Trigger |
|--------|--------|---------|
| `22ad036` | ADD hooks | Users reported "hooks not loading" |
| `a7bc5f2` | REMOVE hooks | Users reported "duplicate hooks error" (#52) |
| `779085e` | ADD hooks | Users reported "agents not loading" (#88) |
| `e3a1306` | REMOVE hooks | Users reported "duplicate hooks error" (#103) |

**Root cause:** Claude Code CLI changed behavior between versions:
- Pre-v2.1: Required explicit declarations
- v2.1+: Auto-loads by convention, errors on duplicate

The same principle now applies to agents, commands, and skills — let auto-discovery handle them all.

### Current Rule (Enforced by Test)

The test `plugin.json does NOT have explicit hooks declaration` in `tests/hooks/hooks.test.js` prevents hooks from being reintroduced.

---

## Minimal Known-Good Example

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My Claude Code plugin"
}
```

That's it. No `agents`, `commands`, `skills`, or `hooks` arrays. Claude Code discovers all components from conventional directory names.

For a more complete metadata example:

```json
{
  "name": "everything-claude-code",
  "version": "1.8.0",
  "description": "Complete collection of battle-tested Claude Code configs...",
  "author": { "name": "...", "url": "..." },
  "homepage": "https://github.com/...",
  "repository": "https://github.com/...",
  "license": "MIT",
  "keywords": ["claude-code", "agents", "skills"]
}
```

---

## Known Anti-Patterns

These look correct but cause problems:

* Adding explicit `agents`, `commands`, or `skills` arrays (bypasses auto-discovery)
* Adding `"hooks": ["./hooks/hooks.json"]` (causes duplicate error)
* String values instead of arrays (if you do declare components)
* Missing `version`
* Relying on inferred paths
* Assuming marketplace behavior matches local validation

**Keep it simple: metadata only, let auto-discovery do the rest.**

---

## Recommendation for Contributors

Before submitting changes that touch `plugin.json`:

1. Keep only metadata fields (name, version, description, author, etc.)
2. Do NOT add component arrays (agents, commands, skills, hooks)
3. Ensure `version` is present
4. Run:

```bash
claude plugin validate .claude-plugin/plugin.json
```

---

## Why This File Exists

This repository is widely forked and used as a reference implementation.

Documenting validator quirks here:

* Prevents repeated issues
* Reduces contributor frustration
* Preserves plugin stability as the ecosystem evolves

If the validator changes, update this document first.
