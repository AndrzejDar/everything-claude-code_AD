# React Orchestrator — Workflow Configuration

This is the default configuration. Override per-project by placing a `workflow-config.md` in your project's `.claude/skills/react-orchestrator/references/`.

## Commands

```yaml
# Quality commands — auto-detected from package.json scripts
lint: npm run lint
lint_fix: npm run lint -- --fix
typecheck: npx tsc --noEmit
build: npm run build
test: npm test
test_coverage: npm test -- --coverage
```

## Coverage Thresholds

```yaml
coverage:
  statements: 80
  branches: 80
  functions: 80
  lines: 80
```

## Skill Triggers

```yaml
skill_triggers:
  mui-expert:
    keywords:
      - MUI
      - Material
      - "@mui/"
      - theme
      - styled
      - DataGrid
      - TextField
      - Dialog
      - Modal
      - Drawer
      - AppBar
      - Snackbar
      - Autocomplete
      - DatePicker
      - Tabs
      - Table
    file_patterns:
      - "**/theme/**"
      - "**/styles/**"

  frontend-patterns:
    keywords:
      - layout
      - state management
      - SSR
      - server component
      - client component
      - routing
      - data fetching
      - SWR
      - React Query
      - Zustand
      - Redux
    file_patterns:
      - "**/pages/**"
      - "**/layouts/**"
      - "**/stores/**"
```

## Retry Configuration

```yaml
retry:
  max_attempts: 3

  retryable:
    - test_failure
    - lint_error
    - coverage_below_threshold
    - review_critical_found
    - security_critical_found

  non_retryable:
    - typecheck_error_after_build_resolver
    - missing_dependency
    - fatal_error
```

## Parallel Execution

```yaml
parallel:
  max_concurrent_tasks: 3

  sequential_patterns:
    - "**/theme/**"
    - "**/contexts/**"
    - "**/App.tsx"
    - "**/index.tsx"
    - "**/package.json"
    - "**/tsconfig.json"
```

## Verification Phases

```yaml
verification:
  phases:
    - name: build
      command: npm run build
      blocking: true

    - name: typecheck
      command: npx tsc --noEmit
      blocking: true

    - name: lint
      command: npm run lint
      blocking: true
      auto_fix: npm run lint -- --fix

    - name: tests
      command: npm test -- --coverage
      blocking: true

    - name: security
      checks:
        - npm audit --audit-level=high
        - grep for hardcoded secrets
        - grep for console.log with sensitive data
      blocking: true

    - name: diff_review
      command: git diff --stat
      blocking: false
```
