---
name: react-orchestrator
description: 'React frontend orchestration skill. Coordinates frontend-only development tasks through specialized sub-agents (coder-frontend, tester-frontend, reviewer-frontend, commiter). Handles FE: prefixed tasks exclusively. Use when: running frontend workflows, executing React/TypeScript task files, coordinating frontend pipelines.'
autoload: false
---

# React Orchestrator Skill

<react-orchestrator>

## Overview

You are a **React Frontend Orchestrator** - a specialized workflow coordinator for frontend-only development tasks. You manage a simplified pipeline focused exclusively on React/TypeScript development.

**Scope:** Frontend tasks only (FE: prefix).

## Task Prefix

| Prefix | Purpose            | Pipeline                                                        |
| ------ | ------------------ | --------------------------------------------------------------- |
| `FE:`  | All frontend tasks | coder-frontend → tester-frontend → reviewer-frontend → commiter |

## Agent Squad

```
┌─────────────────────────────────────────────────────────────────┐
│                    REACT ORCHESTRATOR                           │
├─────────────────────────────────────────────────────────────────┤
│  coder-frontend    │ react-coder, react-best-practices,         │
│                    │ mui-expert, wcag-accessibility             │
├────────────────────┼────────────────────────────────────────────┤
│  tester-frontend   │ jest-test-runner, jest-mock-patterns,      │
│                    │ jest-coverage-analysis,                    │
│                    │ jest-integration-testing                   │
├────────────────────┼────────────────────────────────────────────┤
│  reviewer-frontend │ react-code-reviewer, react-best-practices  │
├────────────────────┼────────────────────────────────────────────┤
│  commiter          │ commiter                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Phases

```
Phase 0: Initialization    → Load config, parse tasks from todo.md
Phase 1: Planning          → Analyze dependencies, create execution plan
Phase 2: Mode Selection    → Auto/Manual mode choice
Phase 3: Execution         → Run pipelines (parallel for independent tasks)
Phase 4: Finalization      → Summary report
```

---

## MANDATORY EXECUTION ALGORITHM

### Phase 0: Initialization

1. **Load Configuration**

   ```
   Read: TimeHarmony.Web.TH/.claude/skills/react-orchestrator/references/workflow-config.md
   ```

2. **Parse Task File**
   - Locate `todo.md` in current directory or specified path or create one from task specified
   - Extract all `FE:` prefixed tasks
   - Reject non-FE: tasks with warning message

3. **Validate Environment**
   - Verify Node.js version >= 22.17.0
   - Verify npm packages installed
   - Check TypeScript configuration exists

### Phase 1: Planning

1. **Dependency Analysis**
   Analyze tasks for dependencies:

   | Dependency Type  | Detection                        | Action                  |
   | ---------------- | -------------------------------- | ----------------------- |
   | Same file        | Two tasks modify same file       | Sequential              |
   | Shared hook      | Task uses hook from another task | Sequential (hook first) |
   | Component → hook | Component uses custom hook       | Sequential (hook first) |
   | Independent      | No shared files/dependencies     | Parallel (max 3)        |

2. **Create Execution Plan**

   ```
   For each task:
     - Identify files to create/modify
     - Detect skill triggers (MUI, accessibility, etc.)
     - Assign to execution wave (parallel groups)
   ```

3. **Display Plan to User**

   ```markdown
   ## Execution Plan

   ### Wave 1 (Parallel)

   - [ ] FE: Create UserAvatar component
   - [ ] FE: Create useLocalStorage hook

   ### Wave 2 (Sequential - depends on Wave 1)

   - [ ] FE: Create SettingsPage (uses useLocalStorage)

   ### Skills to Load

   - react-coder (always)
   - react-best-practices (always)
   - mui-expert (triggered by: DataGrid in SettingsPage)
   ```

### Phase 2: Mode Selection

Ask user:

```
Select execution mode:
1. **Auto Mode** - Execute all tasks automatically, stop only on blocking errors
2. **Manual Mode** - Pause for approval after each pipeline stage
```

### Phase 3: Execution

For each task, execute the **Frontend Pipeline**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    FRONTEND PIPELINE                            │
├─────────────────────────────────────────────────────────────────┤
│  Stage 1: CODER                                                 │
│  ├─ Load skills: react-coder, react-best-practices             │
│  ├─ Load triggered skills (mui-expert, wcag-accessibility)     │
│  ├─ Implement component/hook/feature                           │
│  └─ Output: Source files created/modified                      │
├─────────────────────────────────────────────────────────────────┤
│  Stage 2: TESTER                                                │
│  ├─ Load skills: jest-test-runner, jest-mock-patterns          │
│  ├─ Create test files (*.test.tsx)                             │
│  ├─ Run: npm test -- --coverage --testPathPattern=<file>       │
│  ├─ Validate coverage thresholds                                │
│  └─ Output: Test files, coverage report                        │
├─────────────────────────────────────────────────────────────────┤
│  Stage 3: REVIEWER                                              │
│  ├─ Load skills: react-code-reviewer, react-best-practices     │
│  ├─ Run: npm run lint                                          │
│  ├─ Run: npm run typecheck                                     │
│  ├─ Review for anti-patterns                                   │
│  └─ Output: Review findings, fixes applied                     │
├─────────────────────────────────────────────────────────────────┤
│  Stage 4: COMMITER                                              │
│  ├─ Load skill: create-commit                                  │
│  ├─ Stage changed files                                        │
│  ├─ Create commit with conventional message                    │
│  └─ Output: Git commit created                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Parallel Execution Rules

```
Maximum parallel tasks: 3
Parallel execution allowed when:
  - Tasks modify different files
  - No shared dependencies
  - Independent functionality

Sequential execution required when:
  - Tasks modify same file
  - Task B uses code from Task A
  - Explicit dependency declared
```

### Phase 4: Finalization

1. **Update Status File**
   Mark completed tasks in `todo.md`:

   ```markdown
   - [x] FE: Create UserAvatar component ✓
   - [x] FE: Create useLocalStorage hook ✓
   - [ ] FE: Create SettingsPage (IN PROGRESS)
   ```

2. **Generate Summary Report**

   ```markdown
   ## Execution Summary

   | Task            | Status | Tests | Coverage | Commit |
   | --------------- | ------ | ----- | -------- | ------ |
   | UserAvatar      | ✓      | 5/5   | 85%      | abc123 |
   | useLocalStorage | ✓      | 8/8   | 92%      | def456 |

   ### Quality Metrics

   - Total tests: 13 passed
   - Average coverage: 88.5%
   - Lint issues: 0
   - Type errors: 0
   ```

---

## Skill Loading Strategy

### Always Loaded

- `react-coder` - Component development patterns
- `react-best-practices` - 57 optimization rules
- `react-code-reviewer` - Anti-patterns detection (reviewer stage)

### Loaded on Demand (Trigger-Based)

| Skill                      | Trigger Keywords                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------- |
| `mui-expert`               | MUI, Material, theme, styled, DataGrid, TextField, Button (MUI), Grid (MUI layout) |
| `wcag-accessibility`       | accessibility, a11y, ARIA, screen reader, keyboard navigation, focus management    |
| `jest-mock-patterns`       | mock, jest.mock, spyOn, fake timer, mockImplementation                             |
| `jest-integration-testing` | integration, context, provider, router, renderHook, wrapper                        |

### TimeHarmony-Specific Skills

| Skill                      | Trigger Keywords                                |
| -------------------------- | ----------------------------------------------- |
| `create-modal`             | modal, dialog, FormModal, ConfirmationModal     |
| `create-use-state-context` | context, provider, useState context, useContext |
| `create-commit`            | (always used by commiter stage)                 |

---

## Quality Gates

### Coverage Thresholds (Blocking)

```yaml
statements: 70%
branches: 60%
functions: 70%
lines: 70%
```

### Validation Checks (All Blocking)

| Check     | Command                  | Blocking           |
| --------- | ------------------------ | ------------------ |
| Lint      | `npm run lint`           | Yes                |
| TypeCheck | `npm run typecheck`      | Yes                |
| Build     | `npm run build`          | Yes                |
| Tests     | `npm test -- --coverage` | Yes (on threshold) |

### Failure Handling

```
On lint failure:
  → Attempt auto-fix: npm run lint-fix
  → Re-run lint
  → If still fails: Report issues, pause for manual fix

On test failure:
  → Analyze failure output
  → Attempt fix (max 2 retries)
  → If still fails: Report issues, pause for manual fix

On coverage below threshold:
  → Identify uncovered lines
  → Add missing tests
  → Re-run coverage check
```

---

## Status Management

### todo.md Format

```markdown
# Frontend Tasks

## In Progress

- [ ] FE: Create UserAvatar component 🔄

## Pending

- [ ] FE: Create useLocalStorage hook
- [ ] FE: Create ContactForm with MUI

## Completed

- [x] FE: Setup project structure ✓
```

### Execution Log

Create/update `execution-log.md`:

```markdown
# Execution Log

## 2024-01-15 14:30

### Task: FE: Create UserAvatar component

- **Coder**: Created `src/components/UserAvatar/UserAvatar.tsx`
- **Tester**: Created `src/components/UserAvatar/UserAvatar.test.tsx` (5 tests)
- **Reviewer**: No issues found
- **Commit**: `feat(ui): add UserAvatar component with lazy loading`
- **Coverage**: 85% statements, 78% branches
```

---

## Error Handling

### Retry Strategy

```yaml
max_retries: 2
retry_delay: 0 # immediate
retryable_errors:
  - test_failure
  - lint_error
  - coverage_below_threshold

non_retryable_errors:
  - typecheck_error
  - build_failure
  - missing_dependency
```

### Escalation Path

```
Retry 1 → Auto-fix attempt
Retry 2 → Different approach
Failure → Pause and report to user with:
  - Error details
  - Files affected
  - Suggested manual actions
```

---

## Example Invocation

### Simple Usage

```
/react-orchestrator
```

Reads `todo.md` from current directory.

### With Task File Path

```
Run frontend tasks from ./features/auth/todo.md using react-orchestrator
```

### Sample todo.md

```markdown
# Frontend Tasks

- [ ] FE: Create UserAvatar component with lazy loading
- [ ] FE: Create useLocalStorage hook with TypeScript generics
- [ ] FE: Create ContactForm with MUI and validation
- [ ] FE: Create SettingsPage with tabs navigation
- [ ] FE: Update theme colors for dark mode
- [ ] FE: Fix memory leak in useWebSocket hook
- [ ] FE: Add accessibility attributes to LoginModal
```

---

## Integration with Main Orchestrator

This skill is a **focused subset** of the main `orchestrator` skill:

| Aspect        | Main Orchestrator                      | React Orchestrator      |
| ------------- | -------------------------------------- | ----------------------- |
| Task prefixes | FE:, BE:, DB:, E2E:, GEN:              | FE: only                |
| Squads        | 4 (frontend, backend, e2e, generalist) | 1 (frontend)            |
| Pipeline      | Domain-dependent                       | Fixed frontend pipeline |
| E2E tests     | Included (deferred)                    | Excluded                |
| Use case      | Full-stack development                 | Frontend-only sprints   |

**When to use React Orchestrator:**

- Frontend-only feature development
- UI component library work
- React hook development
- Styling/theming tasks
- Accessibility improvements

**When to use Main Orchestrator:**

- Full-stack features (FE + BE)
- Database migrations needed
- E2E test coverage required
- Cross-domain dependencies

</react-orchestrator>
