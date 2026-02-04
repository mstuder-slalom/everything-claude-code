---
description: Verify implementation against OpenSpec specs using trophy testing methodology. Generates integration-focused tests from WHEN/THEN scenarios AFTER code is written.
---

# Trophy Command

This command invokes the **trophy-guide** agent to verify implementations using trophy testing methodology.

## What This Command Does

1. **Ingest OpenSpec Artifacts** - Read spec.md files with WHEN/THEN scenarios
2. **Ingest Code + Code Map** - Understand implementation structure
3. **Verify Dependencies** - Use tree-sitter to confirm linkages
4. **Generate Integration Tests** - Create tests from WHEN/THEN scenarios
5. **Run Tests** - Verify the build is correct
6. **Report Results** - Show which scenarios pass/fail

## When to Use

Use `/trophy` when:
- Implementation is **complete** and needs verification
- You have OpenSpec artifacts with WHEN/THEN scenarios
- You want to verify behavior against specifications
- You need integration-focused test coverage
- You're validating that a build works correctly

## How It Works

The trophy-guide agent will:

1. **Find OpenSpec artifacts** in `openspec/changes/<change>/specs/`
2. **Extract WHEN/THEN scenarios** from spec.md files
3. **Read source code** and CODEMAP.md for context
4. **Use tree-sitter** (if available) to verify dependency chains
5. **Generate integration tests** for each scenario
6. **Run tests** and report which scenarios pass/fail

## Trophy Testing Philosophy

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

**Key Principles:**
- Implementation FIRST, tests verify it
- Integration tests are PRIMARY
- Minimal mocking - test real behavior
- Quality over coverage numbers

## Example Usage

```
User: /trophy

Agent (trophy-guide):
# Trophy Test Session

## Step 1: Ingesting OpenSpec Artifacts

Found spec files:
- openspec/changes/doc-processing/specs/processor/spec.md
- openspec/changes/doc-processing/specs/exporter/spec.md
- openspec/changes/doc-processing/specs/coordinator/spec.md

Extracted 15 WHEN/THEN scenarios.

## Step 2: Reading Implementation

Source files:
- doc_ingestion/processor.py
- doc_ingestion/exporter.py
- doc_ingestion/coordinator.py

CODEMAP.md loaded - 6 files, 4 entry points.

## Step 3: Generating Integration Tests

Created tests/integration/test_processor.py:
- test_pptx_file_detected()
- test_pdf_file_detected()
- test_unsupported_file_raises_error()
...

## Step 4: Running Tests

pytest tests/integration/ -v

✅ 12 passed
❌ 3 failed

## Trophy Test Report

| Spec File | Scenarios | Passed | Failed |
|-----------|-----------|--------|--------|
| processor/spec.md | 5 | 4 | 1 |
| exporter/spec.md | 3 | 3 | 0 |
| coordinator/spec.md | 7 | 5 | 2 |
| **Total** | **15** | **12** | **3** |

### Failed Scenarios
1. processor/spec.md - Unsupported file type
2. coordinator/spec.md - Batch processing timeout
3. coordinator/spec.md - Error recovery
```

## Test Types Generated

### Integration Tests (PRIMARY)

For each WHEN/THEN scenario:

```python
def test_pptx_file_detected():
    """
    Scenario: PPTX file detected
    WHEN an input file has a .pptx extension
    THEN the system uses PPTX processing logic
    """
    test_file = Path("tests/fixtures/sample.pptx")
    result = detect_document_type(test_file)
    assert result == "pptx"
```

### Unit Tests (SELECTIVE)

Only for complex pure functions not covered by integration:

```python
def test_calculate_similarity_score():
    """Unit test for complex math function"""
    score = calculate_similarity([0.1, 0.2], [0.1, 0.2])
    assert score == 1.0
```

### E2E Tests (CRITICAL PATHS ONLY)

Only if specs identify critical user journeys:

```python
def test_end_to_end_document_processing():
    """E2E test for critical user journey"""
    response = client.post("/upload", files={"doc": test_file})
    assert response.status_code == 200
```

## What Makes This Different from TDD

| Aspect | TDD | Trophy Testing |
|--------|-----|----------------|
| When to test | BEFORE implementation | AFTER implementation |
| Test focus | Unit tests primary | Integration tests primary |
| Mocking | Heavy mocking | Minimal mocking |
| Coverage goal | 80%+ lines | Spec scenario coverage |
| Test source | Developer intuition | OpenSpec WHEN/THEN |

## Integration with Other Commands

- Use `/plan` to understand what to build
- Use code implementation to write the feature
- Use `/trophy` to verify implementation works
- Use `/code-review` to review implementation quality
- Use `/deps` to analyze dependency chains

## Related Agents

This command invokes the `trophy-guide` agent located at:
`agents/trophy-guide.md`

And can reference the `trophy-workflow` skill at:
`skills/trophy-workflow/SKILL.md`

## Prerequisites

For full functionality:
- OpenSpec artifacts in `openspec/changes/<change>/specs/`
- CODEMAP.md at project root (optional but recommended)
- Tree-sitter MCP server configured (optional but recommended)
