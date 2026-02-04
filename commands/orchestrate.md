# Orchestrate Command

Sequential agent workflow for complex tasks.

## Usage

`/orchestrate [workflow-type] [task-description]`

## Workflow Types

### feature
Full feature implementation workflow:
```
planner -> trophy-guide -> code-reviewer -> security-reviewer
```

### bugfix
Bug investigation and fix workflow:
```
explorer -> trophy-guide -> code-reviewer
```

### refactor
Safe refactoring workflow:
```
architect -> code-reviewer -> trophy-guide
```

### security
Security-focused review:
```
security-reviewer -> code-reviewer -> architect
```

### verify
Verification workflow with dependency analysis:
```
deps -> trophy-guide
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
   - Output: `HANDOFF: planner -> trophy-guide`

2. **Trophy Guide Agent**
   - Reads planner handoff and OpenSpec artifacts
   - Generates integration tests from WHEN/THEN scenarios
   - Runs tests to verify implementation
   - Output: `HANDOFF: trophy-guide -> code-reviewer`

3. **Code Reviewer Agent**
   - Reviews implementation
   - Checks for issues
   - Suggests improvements
   - Output: `HANDOFF: code-reviewer -> security-reviewer`

4. **Security Reviewer Agent**
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
Agents: planner -> trophy-guide -> code-reviewer -> security-reviewer

SUMMARY
-------
[One paragraph summary]

AGENT OUTPUTS
-------------
Planner: [summary]
Trophy Guide: [summary]
Code Reviewer: [summary]
Security Reviewer: [summary]

FILES CHANGED
-------------
[List all files modified]

TEST RESULTS
------------
[Trophy test scenario pass/fail summary]

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
- `verify <description>` - Dependency analysis + trophy testing
- `custom <agents> <description>` - Custom agent sequence

## Custom Workflow Example

```
/orchestrate custom "architect,trophy-guide,code-reviewer" "Redesign caching layer"
```

## Tips

1. **Start with planner** for complex features
2. **Always include code-reviewer** before merge
3. **Use security-reviewer** for auth/payment/PII
4. **Keep handoffs concise** - focus on what next agent needs
5. **Use /deps** before /trophy for dependency context
