# Why Trophy Testing?

A guide for developers on what trophy testing is, why it matters, and how to use it.

---

## The Problem with Traditional Testing

### TDD's Promise vs Reality

**What TDD promises:** Write tests first, then code to make them pass. This ensures everything is tested.

**What actually happens:**
1. Developer writes test for function they're about to create
2. Test uses mocks because dependencies don't exist yet
3. Developer writes code to make mocked test pass
4. Mocked test passes, developer moves on
5. **Real integration is never tested**
6. Production breaks because mocks don't reflect reality

### The Mocking Trap

```python
# This test passes but proves nothing
def test_user_service():
    mock_db = Mock()
    mock_db.get_user.return_value = {"id": 1, "name": "Test"}

    service = UserService(mock_db)
    user = service.get_user(1)

    assert user["name"] == "Test"  # ✅ Passes!
    # But does the real database query work? Who knows.
```

**The mock test verifies:** "If the database returned this exact data, would my code handle it?"

**What you actually need to know:** "Does my code work with the real database?"

---

## Trophy Testing: A Different Approach

### The Core Insight

**Don't test whether mocks work. Test whether your code works.**

Trophy testing inverts the traditional approach:

| Traditional TDD | Trophy Testing |
|-----------------|----------------|
| Write test first | Write specification first |
| Mock dependencies | Use real dependencies |
| Tests drive design | Specs drive design, tests verify |
| Unit tests primary | Integration tests primary |
| High mock coverage | High behavior coverage |

### The Trophy Shape

```
       /\
      /E2E\        ← 10%: Critical user journeys only
     /------\
    /INTEGR. \     ← 70%: PRIMARY FOCUS - Real behavior
   /----------\
  /   UNIT    \    ← 20%: Complex pure functions only
 /--------------\
|    STATIC     |  ← Free: TypeScript, ESLint, go vet
------------------
```

**Why this shape?**
- **Integration tests catch real bugs** - They test actual behavior
- **Unit tests are limited** - They only catch bugs in isolated logic
- **E2E tests are expensive** - Slow, flaky, hard to maintain
- **Static analysis is free** - TypeScript/ESLint catch errors at compile time

---

## The Trophy Workflow

### Step 1: Write Specifications (Not Tests)

Before writing code, define **what** the system should do using OpenSpec:

```markdown
# Authentication Specification

### Requirement: User Login

#### Scenario: Valid credentials
- **WHEN** user submits correct email and password
- **THEN** system returns JWT token
- **AND** token expires in 24 hours

#### Scenario: Invalid password
- **WHEN** user submits wrong password
- **THEN** system returns 401 Unauthorized
- **AND** error message says "Invalid credentials"
```

**Why specs instead of tests?**
- Specs are readable by non-developers
- Specs define behavior, not implementation
- Specs can be reviewed before coding starts
- Specs become the source of truth for tests

### Step 2: Implement from Spec

Write the code based on the specification. The spec tells you:
- What inputs to handle
- What outputs to produce
- What edge cases exist

### Step 3: Generate Tests from Spec

Each WHEN/THEN scenario becomes an integration test:

```python
def test_valid_credentials_returns_jwt():
    """
    Scenario: Valid credentials
    WHEN user submits correct email and password
    THEN system returns JWT token
    """
    # Arrange - real database, real user
    db = get_test_database()
    create_user(db, email="test@example.com", password="correct")

    # Act - real HTTP call
    response = client.post("/login", json={
        "email": "test@example.com",
        "password": "correct"
    })

    # Assert - verify THEN conditions
    assert response.status_code == 200
    assert "token" in response.json()

def test_invalid_password_returns_401():
    """
    Scenario: Invalid password
    WHEN user submits wrong password
    THEN system returns 401 Unauthorized
    """
    db = get_test_database()
    create_user(db, email="test@example.com", password="correct")

    response = client.post("/login", json={
        "email": "test@example.com",
        "password": "wrong"
    })

    assert response.status_code == 401
    assert response.json()["error"] == "Invalid credentials"
```

### Step 4: Run Tests to Verify Build

The tests prove the implementation matches the specification:

```
Trophy Test Report
==================
auth/spec.md: 5/5 scenarios ✅
users/spec.md: 8/8 scenarios ✅
payments/spec.md: 6/7 scenarios ⚠️

Failed: payments/spec.md - "Refund exceeds original amount"
  Expected: 400 error
  Actual: 500 internal server error
```

---

## The Critical Rule: No Internal Mocking

**Tests that mock internal project code are NOT trophy tests.**

This is the most important rule. If you mock your own code, you're not testing whether your code works.

### What You Can Mock

| Mock This | Why |
|-----------|-----|
| Stripe API | External, costs money |
| OpenAI API | External, costs money |
| Twilio SMS | External, rate limited |
| Third-party services | Not in your control |

### What You Cannot Mock

| Never Mock This | Use This Instead |
|-----------------|------------------|
| Your database | Test database, SQLite in-memory, Testcontainers |
| Your HTTP handlers | Test client (TestClient, httptest) |
| Your services | Call them directly |
| Your internal modules | Test them together |

### Why This Matters

```
❌ Mock test passes → Mocks work correctly → Real code might be broken
✅ Real test passes → Real code works → Production will work
```

---

## Tools in This Repo

### Commands

| Command | Purpose |
|---------|---------|
| `/trophy` | Generate and run trophy tests from specs |
| `/trophy-workflow` | Full guided workflow with language detection |
| `/deps` | Analyze code dependencies with tree-sitter |
| `/go-trophy` | Go-specific trophy testing |

### Agents

| Agent | Purpose |
|-------|---------|
| `trophy-guide` | Orchestrates trophy test generation |
| `planner` | Creates implementation plans from requirements |
| `code-reviewer` | Reviews code quality after implementation |

### Supporting Tools

| Tool | Purpose |
|------|---------|
| `mcp-tree-sitter` | Analyzes code dependencies for accurate testing |
| `/cartographer` | Maps codebase structure |
| `/deps` | Validates dependency chains |

---

## Language Support

Trophy testing adapts to your project's language:

| Language | Test Pattern | Framework |
|----------|--------------|-----------|
| Go | Table-driven tests | `go test` |
| Python | pytest fixtures | `pytest` |
| TypeScript | describe/it blocks | Jest/Vitest |
| Java | JUnit 5 + Testcontainers | Maven/Gradle |

---

## Quick Start

### 1. Install Components

```bash
git clone https://github.com/mstuder-slalom/everything-claude-code.git
cp -r everything-claude-code/rules/* ~/.claude/rules/
cp -r everything-claude-code/agents/* ~/.claude/agents/
cp -r everything-claude-code/commands/* ~/.claude/commands/
```

### 2. Install Tree-Sitter MCP

```bash
git clone https://github.com/mstuder-slalom/mcp-tree-sitter.git
cd mcp-tree-sitter && pip install -e .
```

### 3. Run Trophy Workflow

```bash
# In your project
/trophy-workflow
```

---

## Comparison: Before and After

### Before (TDD with Mocks)

```python
# test_user_service.py - 50 lines of mock setup
@patch('services.user.database')
@patch('services.user.cache')
@patch('services.user.email_service')
def test_create_user(mock_email, mock_cache, mock_db):
    mock_db.insert.return_value = {"id": 1}
    mock_cache.set.return_value = True
    mock_email.send.return_value = True

    result = create_user({"name": "Test"})

    assert result["id"] == 1
    mock_db.insert.assert_called_once()
    # Tests that mocks were called, not that code works
```

**Problems:**
- Tests pass even if database schema is wrong
- Tests pass even if cache key format is wrong
- Tests pass even if email template is broken
- 50 lines of mock setup, 3 lines of actual test

### After (Trophy Testing)

```python
# test_user_integration.py - Real behavior
def test_create_user(test_db, test_cache):
    """
    Scenario: Create new user
    WHEN valid user data is submitted
    THEN user is created in database
    AND welcome email is queued
    """
    result = create_user(test_db, {"name": "Test", "email": "test@example.com"})

    # Verify real database
    assert test_db.get_user(result["id"]) is not None

    # Verify real cache
    assert test_cache.get(f"user:{result['id']}") is not None

    # Verify real email queue (or mock external email API only)
    assert email_queue.pending_count() == 1
```

**Benefits:**
- Tests fail if database schema is wrong
- Tests fail if cache key format is wrong
- Tests catch real integration bugs
- Clear connection to spec scenarios

---

## FAQ

### Q: Isn't this just integration testing?

Yes, but with structure. Trophy testing provides:
- Specs that define what to test (WHEN/THEN scenarios)
- Clear rules about what to mock (nothing internal)
- A workflow that connects specs → implementation → tests
- Language-specific patterns for each ecosystem

### Q: What about test speed?

Integration tests are slower than unit tests, but:
- They catch more bugs per test
- You write fewer tests overall
- Use test databases (SQLite in-memory is fast)
- Parallelize where possible

### Q: When should I write unit tests?

Only for complex pure functions:
- Mathematical calculations
- Data transformations with many edge cases
- Algorithms with clear inputs/outputs

If integration tests already cover it, skip the unit test.

### Q: How do I handle external APIs?

Mock them. External APIs (Stripe, OpenAI, etc.) are the exception because:
- You don't control them
- They cost money
- They have rate limits
- They may be unavailable

### Q: What if my code is hard to test without mocks?

This usually indicates tight coupling. The solution is to refactor the code, not to add mocks. Trophy testing surfaces design problems early.

---

## Summary

| Principle | Description |
|-----------|-------------|
| **Specs First** | Define behavior in WHEN/THEN scenarios before coding |
| **Implementation Second** | Write code from spec, not from tests |
| **Tests Verify** | Tests prove the implementation matches the spec |
| **No Internal Mocking** | Never mock your own code - test real behavior |
| **Integration Primary** | 70%+ of tests should be integration tests |

Trophy testing answers the question: **"Does my code actually work?"**

Not "Do my mocks return what I expect?" but "Does the real system behave correctly?"

---

## Next Steps

1. Read the [QUICKSTART-TROPHY.md](../QUICKSTART-TROPHY.md) for setup instructions
2. Try `/trophy-workflow` on an existing project
3. Review the [agents/trophy-guide.md](../agents/trophy-guide.md) for detailed patterns
4. Check [docs/COMMANDS.md](COMMANDS.md) for all available commands
