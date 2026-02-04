# Commands Reference

Complete reference for all slash commands available in Everything Claude Code.

---

## Trophy Testing Commands

### `/trophy`

**Verify implementation against OpenSpec specs using trophy testing methodology.**

Generates integration-focused tests from WHEN/THEN scenarios AFTER code is written.

**What it does:**
1. Ingest OpenSpec artifacts (spec.md files with WHEN/THEN scenarios)
2. Ingest source code and code map
3. Verify dependencies using tree-sitter
4. Generate integration tests from WHEN/THEN scenarios
5. Run tests and report results

**When to use:**
- After implementing features from OpenSpec specifications
- To verify code behavior matches spec scenarios
- Before committing to ensure implementation is correct

**Example:**
```bash
/trophy
```

---

### `/trophy-workflow`

**Complete end-to-end trophy testing workflow with automatic language detection.**

Maps codebase, analyzes dependencies, creates specs, generates language-specific integration tests, and runs appropriate code review.

**What it does:**
1. Detect project language (Go, Python, TypeScript, Java)
2. Map codebase with `/cartographer`
3. Analyze dependencies with `/deps`
4. Create OpenSpec artifacts
5. Generate and run trophy tests
6. Run language-specific code review

**When to use:**
- Starting trophy testing on a new project
- Full verification workflow from scratch
- Learning the trophy testing process

**Example:**
```bash
/trophy-workflow run
/trophy-workflow run --lang go
/trophy-workflow run --lang python
```

---

### `/go-trophy`

**Go-specific trophy testing with table-driven tests.**

**What it does:**
1. Read OpenSpec WHEN/THEN scenarios
2. Generate Go table-driven integration tests
3. Use real resources (databases, HTTP clients)
4. Run tests with `go test -race`
5. Report scenario pass/fail status

**When to use:**
- Verifying Go implementations against specs
- Generating idiomatic Go tests

**Example:**
```bash
/go-trophy
```

---

## Dependency Analysis Commands

### `/deps`

**Parse source code dependencies using tree-sitter.**

Confirms linkages and validates code map accuracy.

**What it does:**
1. Detect programming language from files
2. Parse source files using tree-sitter grammar
3. Extract imports, exports, function calls
4. Build file and function level dependency graph
5. Validate against CODEMAP.md
6. Report discrepancies

**When to use:**
- After creating or updating CODEMAP.md
- Before trophy testing to verify understanding
- When investigating module relationships

**Example:**
```bash
/deps src/
/deps app/ --language python
/deps ./... --depth full
```

**Output:**
```json
{
  "files": {
    "api/auth.py": {
      "imports": { "external": ["flask"], "internal": ["services.user"] },
      "exports": { "functions": ["login", "logout"] }
    }
  },
  "dependency_graph": { "file_level": {} }
}
```

---

### `/grammar`

**Generate tree-sitter grammar for unsupported languages.**

**What it does:**
1. Analyze sample source files
2. Identify import/export patterns
3. Generate tree-sitter grammar rules
4. Save grammar to `~/.claude/grammars/`

**When to use:**
- Working with languages not supported by default (RPG, COBOL, etc.)
- Custom DSLs that need dependency analysis

**Example:**
```bash
/grammar rpgle /path/to/sample/files
/grammar cobol samples/*.cbl
```

---

## Planning Commands

### `/plan`

**Create implementation plan before coding.**

Restates requirements, assesses risks, and creates step-by-step implementation plan. **WAIT for user CONFIRM before touching any code.**

**What it does:**
1. Analyze requirements thoroughly
2. Break down into manageable phases
3. Identify dependencies and risks
4. Create detailed implementation steps
5. Present plan for approval

**When to use:**
- Before implementing complex features
- When requirements need clarification
- For architectural changes

**Example:**
```bash
/plan "Add user authentication with OAuth"
```

---

### `/orchestrate`

**Run multiple commands in sequence.**

**Modes:**
- `verify` - Run deps + trophy in sequence

**Example:**
```bash
/orchestrate verify
```

---

## Code Review Commands

### `/code-review`

**General quality review for code changes.**

**What it does:**
1. Analyze code for quality issues
2. Check security concerns
3. Review maintainability
4. Suggest improvements

---

### `/go-review`

**Go-specific code review.**

Focuses on idiomatic Go, concurrency patterns, error handling, and performance.

**Example:**
```bash
/go-review
```

---

### `/python-review`

**Python-specific code review.**

Focuses on PEP 8 compliance, type hints, Pythonic idioms, and security.

**Example:**
```bash
/python-review
```

---

## Build & Fix Commands

### `/build-fix`

**Fix build errors incrementally.**

**What it does:**
1. Analyze error messages
2. Identify root causes
3. Apply minimal fixes
4. Verify build succeeds

---

### `/go-build`

**Fix Go build errors, go vet warnings, and linter issues.**

Invokes the go-build-resolver agent for minimal, surgical fixes.

**Example:**
```bash
/go-build
```

---

## Testing Commands

### `/e2e`

**Generate and run end-to-end tests with Playwright.**

Creates test journeys, runs tests, captures screenshots/videos/traces.

---

### `/test-coverage`

**Analyze test coverage.**

---

### `/verify`

**Run verification loop.**

---

### `/checkpoint`

**Save verification state.**

---

## Refactoring Commands

### `/refactor-clean`

**Remove dead code and consolidate duplicates.**

Runs analysis tools (knip, depcheck, ts-prune) to identify dead code.

---

## Documentation Commands

### `/update-codemaps`

**Update CODEMAP.md files.**

---

### `/update-docs`

**Update documentation.**

---

### `/progress-documentation`

**Track and document progress.**

---

## Learning Commands

### `/learn`

**Extract patterns mid-session.**

---

### `/skill-create`

**Generate skills from git history.**

Analyzes local git history to extract coding patterns and generate SKILL.md files.

**Example:**
```bash
/skill-create
/skill-create --instincts
```

---

### `/instinct-status`

**Show all learned instincts with confidence levels.**

---

### `/instinct-import`

**Import instincts from teammates or other sources.**

---

### `/instinct-export`

**Export instincts for sharing.**

---

### `/evolve`

**Cluster related instincts into skills, commands, or agents.**

---

## Configuration Commands

### `/setup-pm`

**Configure package manager.**

Interactive setup for npm, pnpm, yarn, or bun.

**Example:**
```bash
/setup-pm
```

---

### `/eval`

**Run evaluation harness.**

---

## Command Summary Table

| Command | Category | Description |
|---------|----------|-------------|
| `/trophy` | Testing | Verify implementation against specs |
| `/trophy-workflow` | Testing | Complete trophy workflow |
| `/go-trophy` | Testing | Go-specific trophy testing |
| `/deps` | Analysis | Analyze code dependencies |
| `/grammar` | Analysis | Generate tree-sitter grammar |
| `/plan` | Planning | Create implementation plan |
| `/orchestrate` | Workflow | Run commands in sequence |
| `/code-review` | Review | General code review |
| `/go-review` | Review | Go-specific review |
| `/python-review` | Review | Python-specific review |
| `/build-fix` | Build | Fix build errors |
| `/go-build` | Build | Fix Go build errors |
| `/e2e` | Testing | E2E test generation |
| `/test-coverage` | Testing | Coverage analysis |
| `/refactor-clean` | Refactor | Dead code removal |
| `/skill-create` | Learning | Generate skills from git |
| `/instinct-status` | Learning | View learned instincts |
| `/evolve` | Learning | Cluster instincts into skills |
| `/setup-pm` | Config | Configure package manager |
