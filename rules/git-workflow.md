# Git Workflow

## Commit Message Format

```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

Note: Attribution disabled globally via ~/.claude/settings.json.

## Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

## Feature Implementation Workflow

1. **Plan First**
   - Use **planner** agent to create implementation plan
   - Identify dependencies and risks
   - Break down into phases

2. **Implement from Spec**
   - Write code based on OpenSpec specifications
   - Follow the architecture from design.md
   - Complete implementation before testing

3. **Trophy Testing**
   - Use **trophy-guide** agent to verify implementation
   - Generate integration tests from WHEN/THEN scenarios
   - Run tests to validate behavior
   - Fix any failing scenarios

4. **Code Review**
   - Use **code-reviewer** agent immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

5. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format
