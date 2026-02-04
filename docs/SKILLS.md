# Skills Reference

Complete reference for all skills available in Everything Claude Code.

---

## What Are Skills?

Skills are reusable workflow definitions and domain knowledge that Claude can invoke. They provide:
- **Best practices** for specific domains
- **Workflow patterns** for common tasks
- **Testing methodologies** for different languages

---

## Trophy Testing Skills

### trophy-workflow

**Complete end-to-end trophy testing with language detection.**

**Location:** `skills/trophy-workflow/`

**What it provides:**
- Full workflow from codebase mapping to test verification
- Automatic language detection (Go, Python, TypeScript, Java)
- Language-specific test patterns
- Integration with tree-sitter MCP

**Steps:**
1. Detect project language
2. Map codebase with cartographer
3. Analyze dependencies with tree-sitter
4. Create OpenSpec artifacts
5. Implement from spec
6. Generate and run trophy tests
7. Run code review

**Invoke with:** `/trophy-workflow`

---

### django-trophy

**Django-specific trophy testing with pytest-django.**

**Location:** `skills/django-trophy/`

**What it provides:**
- pytest-django integration patterns
- Real database testing (no mocking)
- Django test client usage
- Model factory patterns

**Key Principles:**
- Use `@pytest.mark.django_db` for database access
- Test with real Django ORM, not mocks
- Use `APIClient` for endpoint tests
- Only mock external services (Stripe, etc.)

**Invoke with:** Django is auto-detected by `/trophy-workflow`

---

### springboot-trophy

**Spring Boot trophy testing with Testcontainers.**

**Location:** `skills/springboot-trophy/`

**What it provides:**
- JUnit 5 integration patterns
- Testcontainers for real database testing
- MockMvc for controller tests
- Minimal external service mocking

**Key Principles:**
- Use `@SpringBootTest` for integration tests
- Use Testcontainers for PostgreSQL/MySQL
- Use MockMvc for HTTP endpoint tests
- Only mock external APIs

**Invoke with:** Java/Spring is auto-detected by `/trophy-workflow`

---

### golang-testing

**Go testing patterns with table-driven tests.**

**Location:** `skills/golang-testing/`

**What it provides:**
- Table-driven test patterns
- Subtests with `t.Run()`
- Benchmark patterns
- Fuzzing patterns
- Test coverage strategies

**Key Principles:**
- Use table-driven tests for multiple scenarios
- Use `t.Parallel()` for concurrent tests
- Use `testify` for assertions
- Test with real databases using `testcontainers-go`

**Example:**
```go
func TestLogin(t *testing.T) {
    tests := []struct {
        name    string
        input   LoginRequest
        want    int
        wantErr bool
    }{
        {"valid credentials", validReq, 200, false},
        {"invalid password", invalidReq, 401, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // test implementation
        })
    }
}
```

**Invoke with:** Go is auto-detected by `/trophy-workflow` or `/go-trophy`

---

### python-testing

**Python testing with pytest and integration-first approach.**

**Location:** `skills/python-testing/`

**What it provides:**
- pytest fixture patterns
- Integration test strategies
- Minimal mocking guidelines
- Coverage configuration

**Key Principles:**
- Use pytest fixtures for setup/teardown
- Test with real databases (SQLite in-memory or test DB)
- Use `pytest-asyncio` for async code
- Only mock external HTTP calls

**Invoke with:** Python is auto-detected by `/trophy-workflow`

---

## Code Analysis Skills

### codemap-template

**Generate and maintain CODEMAP.md files.**

**Location:** `skills/codemap-template/`

**What it provides:**
- Template for codebase documentation
- Structure for architecture overview
- Dependency documentation patterns

**Sections:**
- Overview (what the project does)
- Directory structure
- Entry points
- Module dependencies
- Data flow
- External dependencies

**Invoke with:** `/cartographer` or ask to "create a code map"

---

## Language Pattern Skills

### golang-patterns

**Idiomatic Go patterns and best practices.**

**Location:** `skills/golang-patterns/`

**Covers:**
- Error handling patterns
- Concurrency with goroutines/channels
- Interface design
- Package organization
- Performance optimization

---

### python-patterns

**Pythonic idioms and PEP 8 standards.**

**Location:** `skills/python-patterns/`

**Covers:**
- PEP 8 compliance
- Type hints best practices
- Context managers
- Decorators
- Package structure

---

### django-patterns

**Django architecture and REST API patterns.**

**Location:** `skills/django-patterns/`

**Covers:**
- DRF (Django REST Framework) patterns
- ORM best practices
- Caching strategies
- Signal usage
- Middleware patterns

---

### springboot-patterns

**Spring Boot architecture patterns.**

**Location:** `skills/springboot-patterns/`

**Covers:**
- Layered architecture
- REST API design
- Data access patterns
- Caching with Redis
- Async processing
- Logging best practices

---

### backend-patterns

**General backend architecture patterns.**

**Location:** `skills/backend-patterns/`

**Covers:**
- API design principles
- Database optimization
- Server-side best practices
- Node.js/Express patterns
- Next.js API routes

---

### frontend-patterns

**React and Next.js patterns.**

**Location:** `skills/frontend-patterns/`

**Covers:**
- React component patterns
- State management
- Performance optimization
- UI best practices

---

## Security Skills

### security-review

**Comprehensive security checklist and patterns.**

**Location:** `skills/security-review/`

**When to use:**
- Adding authentication
- Handling user input
- Working with secrets
- Creating API endpoints
- Implementing payment features

**Covers:**
- OWASP Top 10
- Input validation
- Authentication patterns
- Secret management
- Rate limiting

---

### django-security

**Django-specific security patterns.**

**Location:** `skills/django-security/`

**Covers:**
- CSRF protection
- SQL injection prevention
- XSS prevention
- Authentication/authorization
- Secure deployment

---

### springboot-security

**Spring Security best practices.**

**Location:** `skills/springboot-security/`

**Covers:**
- Authentication/authorization
- Input validation
- CSRF protection
- Secret management
- Rate limiting
- Security headers

---

## Database Skills

### postgres-patterns

**PostgreSQL optimization and design patterns.**

**Location:** `skills/postgres-patterns/`

**Covers:**
- Query optimization
- Schema design
- Indexing strategies
- Security best practices
- Supabase integration

---

### clickhouse-io

**ClickHouse analytics patterns.**

**Location:** `skills/clickhouse-io/`

**Covers:**
- Query optimization for analytics
- Schema design for time-series
- Data engineering patterns
- High-performance analytics

---

### jpa-patterns

**JPA/Hibernate patterns for Spring Boot.**

**Location:** `skills/jpa-patterns/`

**Covers:**
- Entity design
- Relationship mapping
- Query optimization
- Transaction management
- Auditing
- Pagination

---

## Learning Skills

### continuous-learning

**Auto-extract patterns from sessions.**

**Location:** `skills/continuous-learning/`

**What it does:**
- Observe coding sessions
- Extract reusable patterns
- Save as learned skills

---

### continuous-learning-v2

**Instinct-based learning with confidence scoring.**

**Location:** `skills/continuous-learning-v2/`

**What it does:**
- Create atomic "instincts" from observations
- Score instincts by confidence
- Cluster into skills/commands/agents via `/evolve`
- Support import/export for sharing

**Commands:**
- `/instinct-status` - View learned instincts
- `/instinct-import` - Import from others
- `/instinct-export` - Export for sharing
- `/evolve` - Cluster into skills

---

## Workflow Skills

### iterative-retrieval

**Progressive context refinement for subagents.**

**Location:** `skills/iterative-retrieval/`

**Pattern for solving the "subagent context problem" - helping agents progressively build understanding.**

---

### strategic-compact

**Manual context compaction suggestions.**

**Location:** `skills/strategic-compact/`

**Suggests logical intervals for context compaction to preserve understanding through task phases.**

---

### verification-loop

**Continuous verification methodology.**

**Location:** `skills/verification-loop/`

**Patterns for ongoing verification during development.**

---

### eval-harness

**Formal evaluation framework.**

**Location:** `skills/eval-harness/`

**Eval-driven development (EDD) principles for Claude Code sessions.**

---

## Utility Skills

### coding-standards

**Universal coding standards.**

**Location:** `skills/coding-standards/`

**Covers:**
- TypeScript/JavaScript best practices
- React patterns
- Node.js patterns
- General code quality

---

### progress-documentation

**Progress tracking and documentation.**

**Location:** `skills/progress-documentation/`

**Patterns for documenting progress during development.**

---

## Skills Summary Table

| Category | Skill | Purpose |
|----------|-------|---------|
| Trophy Testing | trophy-workflow | Complete trophy workflow |
| Trophy Testing | django-trophy | Django-specific testing |
| Trophy Testing | springboot-trophy | Spring Boot testing |
| Trophy Testing | golang-testing | Go testing patterns |
| Trophy Testing | python-testing | Python testing patterns |
| Analysis | codemap-template | Generate code maps |
| Patterns | golang-patterns | Go best practices |
| Patterns | python-patterns | Python best practices |
| Patterns | django-patterns | Django patterns |
| Patterns | springboot-patterns | Spring Boot patterns |
| Patterns | backend-patterns | Backend architecture |
| Patterns | frontend-patterns | React/Next.js patterns |
| Security | security-review | Security checklist |
| Security | django-security | Django security |
| Security | springboot-security | Spring Security |
| Database | postgres-patterns | PostgreSQL optimization |
| Database | clickhouse-io | ClickHouse analytics |
| Database | jpa-patterns | JPA/Hibernate |
| Learning | continuous-learning | Pattern extraction |
| Learning | continuous-learning-v2 | Instinct-based learning |
| Workflow | iterative-retrieval | Context refinement |
| Workflow | verification-loop | Continuous verification |
