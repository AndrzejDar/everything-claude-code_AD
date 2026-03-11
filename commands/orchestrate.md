# Orchestrate Command

Sequential agent workflow for complex tasks.

## Usage

`/orchestrate [workflow-type] [task-description]`

## Workflow Types

### feature
Full feature implementation workflow:
```
planner -> tdd-guide -> frontend-test-writer -> code-reviewer -> security-reviewer
```

### feature-frontend
Frontend feature workflow (skips TDD, focuses on component tests after implementation):
```
planner -> code-reviewer -> frontend-test-writer
```

### bugfix
Bug investigation and fix workflow:
```
planner -> tdd-guide -> code-reviewer
```

### bugfix-frontend
Frontend bug fix workflow:
```
planner -> code-reviewer -> frontend-test-writer
```

### refactor
Safe refactoring workflow:
```
architect -> code-reviewer -> tdd-guide
```

### security
Security-focused review:
```
security-reviewer -> code-reviewer -> architect
```

## Execution Pattern

For each agent in the workflow:

1. **Invoke agent** with context from previous agent
2. **Collect output** as structured handoff document
3. **Pass to next agent** in chain
4. **Aggregate results** into final report

## Handoff Document Format

Between agents, create handoff document:

```markdown
## HANDOFF: [previous-agent] -> [next-agent]

### Context
[Summary of what was done]

### Findings
[Key discoveries or decisions]

### Files Modified
[List of files touched]

### Open Questions
[Unresolved items for next agent]

### Recommendations
[Suggested next steps]
```

## Example: Feature Workflow

```
/orchestrate feature "Add user authentication"
```

Executes:

1. **Planner Agent**
   - Analyzes requirements
   - Creates implementation plan
   - Identifies dependencies
   - Output: `HANDOFF: planner -> tdd-guide`

2. **TDD Guide Agent**
   - Reads planner handoff
   - Writes tests first
   - Implements to pass tests
   - Output: `HANDOFF: tdd-guide -> frontend-test-writer`

3. **Frontend Test Writer Agent**
   - Discovers all created/modified frontend files
   - Writes Jest + React Testing Library tests for every component, hook, and utility
   - Ensures 80%+ coverage with proper mocking patterns
   - Runs tests to verify they pass
   - Output: `HANDOFF: frontend-test-writer -> code-reviewer`

4. **Code Reviewer Agent**
   - Reviews implementation AND test code
   - Checks for issues
   - Suggests improvements
   - Output: `HANDOFF: code-reviewer -> security-reviewer`

5. **Security Reviewer Agent**
   - Security audit
   - Vulnerability check
   - Final approval
   - Output: Final Report

## Final Report Format

```
ORCHESTRATION REPORT
====================
Workflow: feature
Task: Add user authentication
Agents: planner -> tdd-guide -> frontend-test-writer -> code-reviewer -> security-reviewer

SUMMARY
-------
[One paragraph summary]

AGENT OUTPUTS
-------------
Planner: [summary]
TDD Guide: [summary]
Frontend Test Writer: [summary]
Code Reviewer: [summary]
Security Reviewer: [summary]

FILES CHANGED
-------------
[List all files modified]

TEST RESULTS
------------
[Test pass/fail summary]

SECURITY STATUS
---------------
[Security findings]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## Parallel Execution

For independent checks, run agents in parallel:

```markdown
### Parallel Phase
Run simultaneously:
- code-reviewer (quality)
- security-reviewer (security)
- architect (design)

### Merge Results
Combine outputs into single report
```

## Arguments

$ARGUMENTS:
- `feature <description>` - Full feature workflow
- `bugfix <description>` - Bug fix workflow
- `refactor <description>` - Refactoring workflow
- `security <description>` - Security review workflow
- `custom <agents> <description>` - Custom agent sequence

## Custom Workflow Example

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## Tips

1. **Start with planner** for complex features
2. **Always include code-reviewer** before merge
3. **Use security-reviewer** for auth/payment/PII
4. **Use frontend-test-writer** after any frontend feature implementation
5. **Use `feature-frontend`** for frontend-only features (skips TDD, adds component tests after implementation)
6. **Keep handoffs concise** - focus on what next agent needs
7. **Run verification** between agents if needed
