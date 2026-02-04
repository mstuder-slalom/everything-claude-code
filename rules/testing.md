# Testing Requirements

## Trophy Testing Methodology

```
       /\
      /E2E\      <- Small: Critical paths only
     /------\
    /INTEGR. \   <- LARGE: Primary focus!
   /----------\
  /   UNIT    \  <- Small: Complex pure functions only
 /--------------\
|    STATIC     | <- Free: TypeScript/ESLint
------------------
```

## Test Distribution

- **Integration Tests** (70%+) - Real behavior with real databases
- **Unit Tests** (20%) - Only for complex pure functions
- **E2E Tests** (10%) - Critical user journeys only

## Trophy Testing Workflow

Implementation-first, then verification:
1. Read OpenSpec specifications
2. Write implementation from spec
3. Generate integration tests from WHEN/THEN scenarios
4. Run tests to verify implementation
5. Report which scenarios pass/fail

## Key Principles

1. **Implementation FIRST** - Code is written from spec, then verified
2. **Spec-driven tests** - WHEN/THEN scenarios become test cases
3. **Integration PRIMARY** - Most tests should be integration tests
4. **NO INTERNAL MOCKING** - This is a hard rule (see below)
5. **Quality over coverage** - Focus on spec scenarios, not line coverage

## CRITICAL: No Internal Mocking

**Tests that mock internal project code are NOT trophy tests.**

This breaks the entire point of trophy testing. If you mock your own code, you're testing whether mocks work, not whether your implementation works.

### NEVER Mock (Hard Rule)

| Internal Code | Instead Use |
|---------------|-------------|
| Your database layer | Test database, SQLite in-memory, Testcontainers |
| Your HTTP handlers | Test client (TestClient, httptest, MockMvc) |
| Your services | Call them directly with real dependencies |
| Your internal modules | Test them together as integration |
| Your file operations | Temp directories, test fixtures |

### ONLY Mock (Exceptions)

| External Code | Why Allowed |
|---------------|-------------|
| Third-party APIs (Stripe, OpenAI, Twilio) | Not in your control, costs money, rate limited |
| External services you don't own | Can't guarantee availability |
| Time/randomness | When determinism is required |

### Why This Matters

```
❌ Mock test passes → Real code is broken → Production fails
✅ Real test passes → Real code works → Production works
```

A test that doesn't execute your real code proves nothing about whether your code works.

## Agent Support

- **trophy-guide** - Use PROACTIVELY to verify implementations against specs
- **e2e-runner** - Playwright E2E testing for critical user flows

## Trophy Test Report

After running tests, expect a report like:

```
Spec Coverage: 12/15 scenarios verified
❌ 3 scenarios failed:
   - processor/spec.md: "Unsupported file type"
   - coordinator/spec.md: "Batch processing timeout"
```
