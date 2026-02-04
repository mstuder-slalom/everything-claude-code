# Everything Claude Code

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Java](https://img.shields.io/badge/-Java-007396?logo=java&logoColor=white)

---

**Production-ready Claude Code configurations featuring Trophy Testing - a spec-driven testing methodology that replaces traditional TDD.**

This repo provides agents, skills, commands, rules, and MCP configurations for building software with Claude Code using integration-focused testing verified against OpenSpec specifications.

---

## What is Trophy Testing?

Trophy Testing is a **spec-driven testing methodology** that inverts traditional TDD:

```
Traditional TDD:        Trophy Testing:
1. Write test           1. Write OpenSpec specification
2. Write code           2. Write implementation from spec
3. Refactor             3. Generate tests from spec scenarios
                        4. Verify implementation matches spec
```

### Why Trophy Testing?

| Problem with TDD | Trophy Solution |
|------------------|-----------------|
| Tests written before understanding requirements | Specs define requirements first |
| Over-mocking leads to false confidence | Integration-focused (70%+ of tests) |
| Tests coupled to implementation details | Tests verify behavior from WHEN/THEN scenarios |
| Hard to know what to test | Spec scenarios become test cases |

### The Trophy Shape

```
       /\
      /E2E\        <- 10%: Critical user journeys only
     /------\
    /INTEGR. \     <- 70%: PRIMARY FOCUS - Real behavior
   /----------\
  /   UNIT    \    <- 20%: Complex pure functions only
 /--------------\
|    STATIC     |  <- Free: TypeScript/ESLint/go vet
------------------
```

---

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/mstuder-slalom/everything-claude-code.git
cd everything-claude-code
```

### 2. Install Rules (Required)

```bash
# Copy rules to your Claude config (applies to all projects)
cp -r rules/* ~/.claude/rules/
```

### 3. Install the Tree-Sitter MCP Server

The MCP server enables code dependency analysis:

```bash
# Install from PyPI
pip install mcp-tree-sitter

# Or from source
git clone https://github.com/mstuder-slalom/mcp-tree-sitter.git
cd mcp-tree-sitter && pip install -e .
```

Add to your Claude settings (`~/.claude/settings.json`):

```json
{
  "mcpServers": {
    "tree-sitter": {
      "command": "python",
      "args": ["-m", "mcp_tree_sitter.server"]
    }
  }
}
```

### 4. Start Using

```bash
# In any project, run the trophy workflow
/trophy-workflow

# Or individual commands
/deps src/           # Analyze dependencies
/trophy              # Run trophy tests
```

---

## Trophy Workflow Overview

The complete workflow transforms specifications into verified, tested code:

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Map Codebase                                       │
│  /cartographer → docs/CODEBASE_MAP.md                       │
├─────────────────────────────────────────────────────────────┤
│  Step 2: Analyze Dependencies                               │
│  /deps src/ → Dependency graph via tree-sitter              │
├─────────────────────────────────────────────────────────────┤
│  Step 3: Create OpenSpec Artifacts                          │
│  proposal.md → design.md → tasks.md → specs/*.md            │
├─────────────────────────────────────────────────────────────┤
│  Step 4: Implement from Spec                                │
│  Write code based on design.md and tasks.md                 │
├─────────────────────────────────────────────────────────────┤
│  Step 5: Generate & Run Trophy Tests                        │
│  /trophy → Tests from WHEN/THEN scenarios                   │
├─────────────────────────────────────────────────────────────┤
│  Step 6: Code Review                                        │
│  Language-specific reviewer (go-reviewer, python-reviewer)  │
└─────────────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
everything-claude-code/
│
├── agents/                 # Specialized subagents
│   ├── trophy-guide.md     # Trophy testing orchestration
│   ├── planner.md          # Implementation planning
│   ├── architect.md        # System design
│   ├── code-reviewer.md    # General code review
│   ├── go-reviewer.md      # Go-specific review
│   ├── python-reviewer.md  # Python-specific review
│   ├── security-reviewer.md
│   ├── build-error-resolver.md
│   ├── go-build-resolver.md
│   ├── e2e-runner.md
│   ├── refactor-cleaner.md
│   ├── doc-updater.md
│   └── database-reviewer.md
│
├── commands/               # Slash commands
│   ├── trophy.md           # /trophy - Run trophy testing
│   ├── trophy-workflow.md  # /trophy-workflow - Full workflow
│   ├── go-trophy.md        # /go-trophy - Go-specific testing
│   ├── deps.md             # /deps - Dependency analysis
│   ├── grammar.md          # /grammar - Generate tree-sitter grammar
│   ├── plan.md             # /plan - Implementation planning
│   ├── orchestrate.md      # /orchestrate - Combined workflows
│   └── ...                 # See docs/COMMANDS.md for full list
│
├── skills/                 # Workflow definitions
│   ├── trophy-workflow/    # Main trophy testing skill
│   ├── django-trophy/      # Django-specific trophy testing
│   ├── springboot-trophy/  # Spring Boot trophy testing
│   ├── golang-testing/     # Go testing patterns
│   ├── python-testing/     # Python testing patterns
│   ├── codemap-template/   # Code map generation
│   └── ...                 # See docs/SKILLS.md for full list
│
├── rules/                  # Always-follow guidelines
│   ├── testing.md          # Trophy testing methodology
│   ├── agents.md           # Agent orchestration
│   ├── git-workflow.md     # Spec-driven workflow
│   ├── coding-style.md     # Immutability, file organization
│   ├── security.md         # Security requirements
│   ├── performance.md      # Model selection
│   ├── hooks.md            # Hook configuration
│   └── patterns.md         # Common patterns
│
├── docs/                   # Documentation
│   ├── TROPHY_WORKFLOW_MAP.md  # Visual workflow architecture
│   ├── COMMANDS.md         # All commands reference
│   ├── SKILLS.md           # All skills reference
│   └── AGENTS.md           # All agents reference
│
├── hooks/                  # Trigger-based automations
├── scripts/                # Cross-platform utilities
├── mcp-configs/            # MCP server configurations
└── examples/               # Example configurations
```

---

## Core Components

### Commands Reference

| Command | Description |
|---------|-------------|
| `/trophy` | Run trophy testing - generates tests from OpenSpec scenarios |
| `/trophy-workflow` | Complete guided workflow with language detection |
| `/deps <path>` | Analyze code dependencies using tree-sitter |
| `/grammar <lang> <path>` | Generate grammar for unsupported languages |
| `/go-trophy` | Go-specific trophy testing with table-driven tests |
| `/plan` | Create implementation plan before coding |
| `/orchestrate verify` | Run deps + trophy in sequence |

### Agents Reference

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| **trophy-guide** | Trophy testing orchestration | Verifying implementations against specs |
| **planner** | Implementation planning | Complex features, architectural decisions |
| **architect** | System design | Major architectural changes |
| **code-reviewer** | General code review | After writing code |
| **go-reviewer** | Go-specific review | Go projects |
| **python-reviewer** | Python-specific review | Python projects |
| **security-reviewer** | Vulnerability analysis | Before commits with auth/input handling |
| **build-error-resolver** | Fix build errors | When builds fail |

### Skills Reference

| Skill | Description |
|-------|-------------|
| **trophy-workflow** | Complete end-to-end trophy testing with language detection |
| **django-trophy** | Django-specific trophy testing with pytest-django |
| **springboot-trophy** | Spring Boot trophy testing with Testcontainers |
| **golang-testing** | Go table-driven tests, benchmarks, fuzzing |
| **python-testing** | pytest fixtures, integration-first approach |
| **codemap-template** | Generate CODEMAP.md for codebase navigation |

---

## Language Support

Trophy testing automatically detects and adapts to your project's language:

| Language | Detection | Test Pattern | Reviewer |
|----------|-----------|--------------|----------|
| Go | `go.mod` | Table-driven tests, subtests | `go-reviewer` |
| Python | `pyproject.toml`, `requirements.txt` | pytest fixtures | `python-reviewer` |
| TypeScript | `package.json` | Jest/Vitest | `code-reviewer` |
| Java | `pom.xml`, `build.gradle` | JUnit 5 + Testcontainers | `code-reviewer` |

### Adding Unsupported Languages

For languages without built-in support (RPG, COBOL, etc.):

```bash
# Generate a tree-sitter grammar from sample files
/grammar rpgle /path/to/sample/files
```

---

## OpenSpec Artifact Structure

Trophy testing uses OpenSpec artifacts to define requirements:

```
openspec/changes/<feature>/
├── .openspec.yaml      # Metadata
├── proposal.md         # What are we building?
├── design.md           # How will we build it?
├── tasks.md            # Step-by-step implementation
└── specs/
    └── <component>/
        └── spec.md     # WHEN/THEN scenarios
```

### Spec Format

```markdown
# Authentication Specification

### Requirement: User Login

#### Scenario: Successful login
- **WHEN** user submits valid email and password
- **THEN** system returns JWT access token
- **AND** token expires in 24 hours

#### Scenario: Invalid credentials
- **WHEN** user submits wrong password
- **THEN** system returns 401 Unauthorized
- **AND** error message says "Invalid credentials"
```

---

## MCP Tree-Sitter Integration

The tree-sitter MCP server provides code analysis capabilities:

### Available Tools

| Tool | Description |
|------|-------------|
| `mcp__tree-sitter__tree-sitter-deps` | Analyze imports, exports, and dependency graphs |
| `mcp__tree-sitter__generate-grammar` | Create grammars for unsupported languages |

### Example Output

```json
{
  "files": {
    "api/auth.py": {
      "imports": {
        "external": ["flask", "jwt"],
        "internal": ["services.user_service"]
      },
      "exports": {
        "functions": ["login", "logout", "verify_token"]
      }
    }
  },
  "dependency_graph": {
    "file_level": {
      "api/auth.py": ["services/user_service.py"]
    }
  }
}
```

---

## Rules (Must Copy Manually)

Claude Code plugins cannot distribute rules automatically. Copy them:

```bash
cp -r rules/* ~/.claude/rules/
```

### Key Rules

- **testing.md** - Trophy testing methodology (70% integration, 20% unit, 10% E2E)
- **agents.md** - When to delegate to specialized agents
- **git-workflow.md** - Spec-driven workflow, commit conventions
- **coding-style.md** - Immutability, file organization (200-400 lines typical)
- **security.md** - Mandatory security checks before commits

---

## Example: Python Flask API

### 1. Create Code Map

```bash
/cartographer
```

Output: `docs/CODEBASE_MAP.md` with architecture overview.

### 2. Analyze Dependencies

```bash
/deps app/
```

Output: JSON showing imports, exports, and dependency graph.

### 3. Create OpenSpec

```
openspec/changes/auth/specs/login/spec.md
```

```markdown
#### Scenario: Valid credentials
- **WHEN** user posts valid email and password
- **THEN** 200 response with JWT token

#### Scenario: Invalid password
- **WHEN** user posts valid email but wrong password
- **THEN** 401 response with "Invalid credentials"
```

### 4. Run Trophy Tests

```bash
/trophy
```

Output:
```
Trophy Test Report
==================
auth/login/spec.md: 2/2 scenarios ✓

All scenarios verified!
```

---

## Troubleshooting

### "Command not found: /trophy"

Ensure the skills are in your Claude config:
```bash
cp -r skills/* ~/.claude/skills/
cp -r commands/* ~/.claude/commands/
```

### "MCP server not available"

Check tree-sitter installation:
```bash
pip show mcp-tree-sitter
```

Verify settings.json has `mcpServers` configured.

### "Language not supported"

Generate a grammar for your language:
```bash
/grammar <language-name> /path/to/sample/files
```

---

## Contributing

Contributions welcome! Areas of interest:

- Language-specific skills (Rust, Ruby, PHP)
- Framework-specific configs (Rails, Laravel, FastAPI)
- Additional MCP integrations
- Documentation improvements

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Related Repositories

- **[mcp-tree-sitter](https://github.com/mstuder-slalom/mcp-tree-sitter)** - MCP server for code dependency analysis
- **[OpenSpec](https://github.com/your-org/openspec)** - Specification format for spec-driven development

---

## License

MIT - Use freely, modify as needed, contribute back if you can.

---

**Read [QUICKSTART-TROPHY.md](QUICKSTART-TROPHY.md) to get started in 5 minutes.**
