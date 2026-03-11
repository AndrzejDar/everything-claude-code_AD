---
description: >
  Run the committer agent to stage and commit current changes.
  Optionally accepts a task context to help build the commit message.
argument-hint: "[task context]"
---

# Run Commit

## Input

The user provided: `$ARGUMENTS`

## Instructions

Use the Task tool to launch the `everything-claude-code:committer` agent:

- **subagent_type**: `everything-claude-code:committer`
- **description**: "Commit current changes"
- **prompt**: Include the following context for the committer agent:
  - **Mode**: standalone (/run-commit)
  - **Context** (if any): `$ARGUMENTS`
  - **Staging mode**: If `$ARGUMENTS` contains a request to commit everything (e.g. "all", "wszystko", "everything"), tell the agent to use `git add -A`. Otherwise, tell the agent to use `git add -u` (default).

## CRITICAL
- DO NOT inform the user what you are doing. Just run the agent — the user invoked this command and knows what they're doing.
