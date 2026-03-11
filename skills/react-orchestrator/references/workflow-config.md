# React Orchestrator - Workflow Configuration

## Project Paths

```yaml
project_root: TimeHarmony.Web.TH
source_dir: src
test_output_dir: .temp/jest
coverage_output_dir: .temp/jest/coverage
```

## Commands

### Development Commands

```yaml
install: npm install
dev: npm run start
build: npm run build
build_dev: npm run build-dev
```

### Quality Commands

```yaml
lint: npm run lint
lint_fix: npm run lint-fix
typecheck: npm run typecheck
prettier: npm run prettier
```

### Test Commands

```yaml
# Run all tests
test_all: npm test

# Run tests with coverage
test_coverage: npm test -- --coverage

# Run specific test file
test_file: npm test -- --testPathPattern={file}

# Run tests matching pattern
test_pattern: npm test -- --testNamePattern="{pattern}"

# Run tests with verbose output
test_verbose: npm test -- --verbose

# Run tests in watch mode (interactive)
test_watch: npm test -- --watch

# Update snapshots
test_update_snapshots: npm test -- --updateSnapshot
```

## Coverage Thresholds

```yaml
coverage:
  statements: 70
  branches: 60
  functions: 70
  lines: 70

  # Per-file thresholds (optional overrides)
  per_file:
    # Complex components may have lower thresholds
    "src/components/**/*.tsx":
      statements: 65
      branches: 55

    # Hooks should have higher coverage
    "src/hooks/**/*.ts":
      statements: 80
      branches: 70
```

## File Patterns

```yaml
patterns:
  # Source files
  components: "src/components/**/*.tsx"
  hooks: "src/hooks/**/*.ts"
  utils: "src/utils/**/*.ts"
  contexts: "src/contexts/**/*.tsx"
  pages: "src/pages/**/*.tsx"

  # Test files
  tests: "**/*.test.{ts,tsx}"
  test_utils: "src/test-utils/**/*.{ts,tsx}"

  # Style files
  styles: "**/*.{css,scss}"

  # Type definitions
  types: "src/types/**/*.ts"
```

## Test Configuration

```yaml
jest:
  # Test environment
  test_environment: jsdom

  # Setup files
  setup_files:
    - src/setupTests.ts

  # Module name mapper (for absolute imports)
  module_paths:
    - src

  # Transform ignore patterns
  transform_ignore:
    - node_modules/(?!(some-esm-package)/)

  # Coverage collection
  collect_coverage_from:
    - "src/**/*.{ts,tsx}"
    - "!src/**/*.d.ts"
    - "!src/**/*.stories.{ts,tsx}"
    - "!src/index.tsx"
    - "!src/vite-env.d.ts"
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
      - Button
      - Grid
      - Box
      - Typography
      - Paper
      - Card
      - Dialog
      - Modal
      - Drawer
      - AppBar
      - Toolbar
      - IconButton
      - Checkbox
      - Radio
      - Select
      - Autocomplete
      - DatePicker
      - TimePicker
      - Snackbar
      - Alert
      - Tooltip
      - Tabs
      - Table
      - TableRow
      - TableCell
    file_patterns:
      - "**/theme/**"
      - "**/styles/**"

  wcag-accessibility:
    keywords:
      - accessibility
      - a11y
      - ARIA
      - aria-
      - screen reader
      - keyboard navigation
      - focus management
      - tab index
      - tabIndex
      - role=
      - alt text
      - label
      - describedby
      - labelledby
    file_patterns:
      - "**/*Modal*"
      - "**/*Dialog*"
      - "**/*Form*"

  jest-mock-patterns:
    keywords:
      - mock
      - jest.mock
      - jest.fn
      - spyOn
      - mockImplementation
      - mockReturnValue
      - mockResolvedValue
      - mockRejectedValue
      - fake timer
      - useFakeTimers
      - advanceTimersByTime
    file_patterns:
      - "**/*.test.*"
      - "**/__mocks__/**"

  jest-integration-testing:
    keywords:
      - integration test
      - context
      - provider
      - router
      - renderHook
      - wrapper
      - QueryClient
      - MemoryRouter
      - ThemeProvider
    file_patterns:
      - "**/*.integration.test.*"
      - "**/test-utils/**"

  create-modal:
    keywords:
      - modal
      - dialog
      - FormModal
      - ConfirmationModal
      - popup
      - overlay
    file_patterns:
      - "**/*Modal*"
      - "**/*Dialog*"

  create-use-state-context:
    keywords:
      - context
      - provider
      - useContext
      - createContext
      - useState context
      - state management
    file_patterns:
      - "**/contexts/**"
      - "**/*Context*"
      - "**/*Provider*"
```

## Retry Configuration

```yaml
retry:
  max_attempts: 3
  delay_ms: 0

  retryable_conditions:
    - exit_code: 1
      stderr_contains: "test failed"
    - exit_code: 1
      stderr_contains: "lint error"
    - exit_code: 1
      stderr_contains: "coverage below threshold"

  non_retryable_conditions:
    - stderr_contains: "TypeScript error"
    - stderr_contains: "Cannot find module"
    - stderr_contains: "Build failed"
    - exit_code: 2  # Fatal error
```

## Parallel Execution

```yaml
parallel:
  max_concurrent_tasks: 3

  # Tasks that should never run in parallel
  sequential_patterns:
    - "**/theme/**"      # Theme changes affect all components
    - "**/contexts/**"   # Context changes have wide impact
    - "**/App.tsx"       # Root component
    - "**/index.tsx"     # Entry point

  # Resource locks (prevent parallel modification)
  resource_locks:
    - pattern: "**/package.json"
      scope: global
    - pattern: "**/tsconfig.json"
      scope: global
```

## Output Formatting

```yaml
output:
  # Status indicators
  status:
    pending: "⏳"
    in_progress: "🔄"
    success: "✓"
    failure: "✗"
    skipped: "⊘"

  # Log format
  log_format: |
    ## {timestamp}

    ### Task: {task_name}
    - **Stage**: {stage}
    - **Status**: {status}
    - **Duration**: {duration}
    - **Output**: {output}

  # Summary table format
  summary_columns:
    - Task
    - Status
    - Tests
    - Coverage
    - Commit
```

## TimeHarmony-Specific Settings

```yaml
timeharmony:
  # Working directory for frontend
  frontend_dir: TimeHarmony.Web.TH

  # API types location (generated from backend)
  api_types_dir: ../TimeHarmony.Web.TSTypes

  # Shared components location
  shared_components:
    - src/components/common
    - src/components/forms
    - src/components/layout

  # Form patterns
  form_components:
    modal: FormModal
    confirmation: ConfirmationModal
    base: BaseForm

  # Context patterns
  context_pattern: create-use-state-context

  # Commit message prefix
  commit_prefix: "feat(ui):"

  # Branch naming
  branch_prefix: "feature/"
```

## Validation Rules

```yaml
validation:
  # Pre-execution checks
  pre_execution:
    - command: node --version
      expected: "v22"
      message: "Node.js 22+ required"

    - command: npm --version
      expected: "10"
      message: "npm 10+ required"

    - file_exists: package.json
      message: "package.json not found"

    - file_exists: tsconfig.json
      message: "tsconfig.json not found"

  # Post-stage checks
  post_coder:
    - no_console_log: true
      exceptions:
        - "console.error"
        - "console.warn"
    - no_any_type: true
      exceptions:
        - "*.d.ts"
    - no_todo_comments: false  # Allow TODOs but report them

  post_tester:
    - coverage_meets_threshold: true
    - no_skipped_tests: true
    - no_focused_tests: true  # No .only() calls

  post_reviewer:
    - lint_passes: true
    - typecheck_passes: true
    - no_unused_imports: true
    - no_unused_variables: true
```

## Error Messages

```yaml
errors:
  coverage_below_threshold: |
    Coverage below required threshold.
    Current: {current}%
    Required: {required}%

    Uncovered lines:
    {uncovered_lines}

    Action: Add tests for uncovered code paths.

  lint_failure: |
    Lint check failed with {error_count} errors.

    {lint_errors}

    Attempting auto-fix...

  typecheck_failure: |
    TypeScript compilation failed.

    {type_errors}

    This is a blocking error. Manual fix required.

  test_failure: |
    {failed_count} test(s) failed.

    {test_errors}

    Analyzing failures...

  build_failure: |
    Build failed.

    {build_errors}

    This is a blocking error. Manual fix required.
```
