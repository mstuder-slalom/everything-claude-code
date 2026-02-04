# Agents Reference

Complete reference for all specialized agents available in Everything Claude Code.

---

## What Are Agents?

Agents are specialized subagents that handle delegated tasks with focused expertise. Each agent has:
- **Specific tools** they can use
- **Model selection** (Opus for complex reasoning, Sonnet for general work)
- **Domain expertise** for particular tasks

---

## Trophy Testing Agent

### trophy-guide

**Trophy Testing specialist that verifies implementations against OpenSpec specs.**

| Property | Value |
|----------|-------|
| Model | Opus |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Purpose:**
- Ingest OpenSpec artifacts (spec.md files with WHEN/THEN scenarios)
- Ingest source code implementation
- Ingest code map (CODEMAP.md) for context
- Use tree-sitter MCP to verify dependency chains
- Generate integration tests that verify WHEN/THEN scenarios
- Run tests to verify implementation correctness

**When to use:**
- PROACTIVELY after code is written
- To verify implementations match specifications
- Before committing feature code

**Trophy Testing Philosophy:**
- Implementation FIRST, then verification
- Integration tests are PRIMARY (70%+)
- Minimal mocking - test real behavior
- Spec scenarios become test cases

---

## Planning Agents

### planner

**Expert planning specialist for complex features and refactoring.**

| Property | Value |
|----------|-------|
| Model | Opus |
| Tools | Read, Grep, Glob |

**Purpose:**
- Analyze requirements and create detailed implementation plans
- Break down complex features into manageable steps
- Identify dependencies and potential risks
- Suggest optimal implementation order
- Consider edge cases and error scenarios

**When to use:**
- PROACTIVELY for complex feature requests
- Before architectural changes
- For multi-file refactoring

---

### architect

**System design specialist for architectural decisions.**

| Property | Value |
|----------|-------|
| Model | Opus |
| Tools | Read, Grep, Glob |

**Purpose:**
- Design system architecture
- Evaluate trade-offs between approaches
- Plan for scalability and maintainability
- Review existing architecture

**When to use:**
- Major system design decisions
- Evaluating technology choices
- Planning large-scale changes

---

## Code Review Agents

### code-reviewer

**General code review for quality, security, and maintainability.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Grep, Glob, Bash |

**Purpose:**
- Review code for quality issues
- Check security concerns
- Evaluate maintainability
- Suggest improvements

**Review Levels:**
- **CRITICAL** - Must fix before merge
- **HIGH** - Should fix
- **MEDIUM** - Consider fixing

**When to use:**
- PROACTIVELY after writing code
- Before committing changes
- For TypeScript/JavaScript projects

---

### go-reviewer

**Go-specific code review specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Grep, Glob, Bash |

**Focus Areas:**
- Idiomatic Go patterns
- Concurrency safety (goroutines, channels, mutexes)
- Error handling patterns
- Performance considerations

**When to use:**
- After writing Go code
- For Go projects specifically

---

### python-reviewer

**Python-specific code review specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Grep, Glob, Bash |

**Focus Areas:**
- PEP 8 compliance
- Type hints and annotations
- Pythonic idioms
- Security best practices

**When to use:**
- After writing Python code
- For Python projects specifically

---

### security-reviewer

**Security vulnerability detection and remediation specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Focus Areas:**
- OWASP Top 10 vulnerabilities
- Secret exposure detection
- Input validation
- Authentication/authorization flaws
- Injection vulnerabilities (SQL, XSS, SSRF)

**When to use:**
- PROACTIVELY after writing auth code
- When handling user input
- Before commits with sensitive data handling

---

### database-reviewer

**PostgreSQL database specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Focus Areas:**
- Query optimization
- Schema design
- Security
- Performance tuning
- Supabase best practices

**When to use:**
- Writing SQL queries
- Creating migrations
- Designing schemas
- Troubleshooting database performance

---

## Build & Error Resolution Agents

### build-error-resolver

**General build error resolution specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Purpose:**
- Analyze build error messages
- Identify root causes
- Apply minimal fixes
- Verify build succeeds

**When to use:**
- When builds fail
- Type errors in TypeScript
- Compilation errors

---

### go-build-resolver

**Go build, vet, and compilation error resolution specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Focus Areas:**
- Go build errors
- `go vet` issues
- Linter warnings
- Minimal, surgical fixes

**When to use:**
- When Go builds fail
- After `go vet` reports issues
- To fix linter warnings quickly

---

## Testing Agents

### e2e-runner

**End-to-end testing specialist using Playwright.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Purpose:**
- Generate E2E tests for critical user flows
- Manage test journeys
- Quarantine flaky tests
- Upload artifacts (screenshots, videos, traces)

**When to use:**
- Critical user journey testing
- Full flow verification
- Visual regression testing

---

## Maintenance Agents

### refactor-cleaner

**Dead code cleanup and consolidation specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Purpose:**
- Run analysis tools (knip, depcheck, ts-prune)
- Identify dead code
- Remove unused exports
- Consolidate duplicates

**When to use:**
- Code maintenance tasks
- Before major refactoring
- Reducing bundle size

---

### doc-updater

**Documentation and codemap specialist.**

| Property | Value |
|----------|-------|
| Model | - |
| Tools | Read, Write, Edit, Bash, Grep, Glob |

**Purpose:**
- Update CODEMAP.md files
- Generate documentation
- Keep READMEs current

**When to use:**
- After significant code changes
- When documentation is outdated

---

## Agent Selection Guide

| Task | Primary Agent | Alternative |
|------|---------------|-------------|
| Verify implementation | trophy-guide | - |
| Plan feature | planner | architect |
| System design | architect | planner |
| Review Go code | go-reviewer | code-reviewer |
| Review Python code | python-reviewer | code-reviewer |
| Review other code | code-reviewer | - |
| Security audit | security-reviewer | code-reviewer |
| Fix build errors | build-error-resolver | go-build-resolver |
| Fix Go builds | go-build-resolver | build-error-resolver |
| E2E testing | e2e-runner | - |
| Remove dead code | refactor-cleaner | - |
| Update docs | doc-updater | - |
| Database review | database-reviewer | - |

---

## Using Agents

Agents are automatically invoked by commands or can be called via the Task tool:

```markdown
# Via commands
/trophy          # Invokes trophy-guide
/go-review       # Invokes go-reviewer
/plan            # Invokes planner

# Directly in conversation
"Use the security-reviewer agent to audit this code"
```

Agents run with their specified tools and model, returning results to the main conversation.
