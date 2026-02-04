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
4. **Minimal mocking** - Test real behavior, not mock behavior
5. **Quality over coverage** - Focus on spec scenarios, not line coverage

## What NOT to Mock

- Your own database - use test database
- Your own HTTP endpoints - use test client
- Your own file system - use temp directories
- Internal modules - test them together

## What TO Mock

- External APIs (Stripe, OpenAI, etc.)
- Third-party services you don't control
- Time/randomness when needed for determinism

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
