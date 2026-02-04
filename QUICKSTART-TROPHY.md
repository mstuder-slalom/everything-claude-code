# Trophy Testing Quick Start Guide

Step-by-step guide to set up and run trophy testing on any codebase.

## Installation (5 minutes)

### Step 1: Install the Plugin

```bash
# Clone everything-claude-code to your Claude plugins directory
git clone https://github.com/affaan-m/everything-claude-code ~/.claude/plugins/everything-claude-code

# Or if you have it locally, copy it
cp -r /path/to/everything-claude-code ~/.claude/plugins/
```

### Step 2: Install the Tree-sitter MCP Server

The MCP server is required for dependency analysis. Install it:

```bash
# Option A: Install from the local repo (recommended during development)
cd /path/to/mcp-tree-sitter
pip install -e .

# Option B: When published to PyPI
pip install mcp-tree-sitter
```

### Step 3: Configure Claude Code Settings

Add the MCP server to your Claude Code configuration.

**Global settings** (`~/.claude/settings.json`):
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

**Or per-project** (`.claude/settings.json` in your repo):
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

### Step 4: Restart Claude Code

Close and reopen Claude Code to load the plugin and MCP server.

### Step 5: Verify Installation

In Claude Code, run:
```
/trophy
```

If you see "trophy-guide" referenced, the plugin is working.

---

## Running Trophy Testing on a New Repo

### Quick Version (Copy-Paste Commands)

```bash
# Navigate to your project
cd /path/to/your/project

# Start Claude Code
claude

# Then in Claude Code, run these commands in order:
```

```
1. Create a CODEMAP.md for this project

2. /deps src/

3. Create OpenSpec artifacts for this project in openspec/changes/

4. /trophy
```

---

### Detailed Walkthrough

#### Phase 1: Create Code Map

Ask Claude to analyze your codebase:

```
Analyze this codebase and create a CODEMAP.md at the project root.
Include:
- Overview (what does this project do?)
- Directory structure (key files only)
- Entry points (main functions, CLI commands, API endpoints)
- Key dependencies between modules
- Data flow (how does data move through the system?)
- External dependencies (npm packages, pip packages, etc.)
```

**Expected output**: A `CODEMAP.md` file at your project root.

#### Phase 2: Analyze Dependencies

Run dependency analysis to verify the code map:

```
/deps src/
```

Or for a specific language:
```
/deps src/ --language python
```

**Expected output**: JSON showing imports, exports, and dependency graph.

#### Phase 3: Create OpenSpec Artifacts

Create the spec directory structure:

```bash
mkdir -p openspec/changes/initial/specs
```

Ask Claude to create specs:

```
Based on the CODEMAP.md and dependency analysis, create OpenSpec spec.md files
for each major component. Use WHEN/THEN scenarios to describe expected behavior.
Put them in openspec/changes/initial/specs/{component}/spec.md
```

**Expected output**: Spec files with WHEN/THEN scenarios like:

```markdown
# Authentication Specification

### Requirement: User Login

#### Scenario: Successful login
- **WHEN** user submits valid credentials
- **THEN** system returns access token
```

#### Phase 4: Generate and Run Trophy Tests

```
/trophy
```

**Expected output**:
- Integration tests generated for each scenario
- Test execution results
- Report showing which scenarios pass/fail

---

## Example: Python Flask API

Here's a complete example for a Python Flask API:

### 1. Code Map Created

```markdown
# Code Map: user-api

## Overview
REST API for user management with authentication.

## Structure
```
app/
├── main.py          # Flask app entry
├── routes/
│   ├── auth.py      # Auth endpoints
│   └── users.py     # User CRUD
├── services/
│   └── user_service.py
└── models/
    └── user.py
```

## Entry Points
- `python -m app.main` - Start server
- `POST /auth/login` - Authentication

## Key Dependencies
- routes/auth.py → services/user_service.py
- services/user_service.py → models/user.py
```

### 2. Dependency Analysis Output

```
/deps app/
```

```json
{
  "files": {
    "routes/auth.py": {
      "imports": {
        "external": ["flask"],
        "internal": ["services.user_service"]
      }
    }
  }
}
```

### 3. OpenSpec Created

`openspec/changes/initial/specs/auth/spec.md`:

```markdown
# Auth Specification

### Requirement: Login

#### Scenario: Valid credentials
- **WHEN** user posts valid email and password
- **THEN** 200 response with JWT token

#### Scenario: Invalid password
- **WHEN** user posts valid email but wrong password
- **THEN** 401 response with "Invalid credentials"
```

### 4. Trophy Tests Generated

```
/trophy
```

Output:
```
Trophy Test Report
==================
auth/spec.md: 2/2 scenarios ✅
users/spec.md: 4/4 scenarios ✅

All scenarios verified!
```

---

## Troubleshooting

### "Command not found: /trophy"

The plugin isn't loaded. Check:
1. Plugin is in `~/.claude/plugins/everything-claude-code/`
2. `plugin.json` exists and is valid JSON
3. Restart Claude Code

### "MCP server not available"

The tree-sitter server isn't configured. Check:
1. `mcp-tree-sitter` is installed: `pip show mcp-tree-sitter`
2. Settings.json has the `mcpServers` configuration
3. Restart Claude Code

### "No spec files found"

Ensure specs are in the correct location:
```
openspec/changes/{feature}/specs/{component}/spec.md
```

### "Language not supported"

For non-standard languages (RPG, COBOL), generate a grammar first:
```
/grammar rpgle /path/to/sample/files
```

---

## Command Reference

| Command | What it does |
|---------|-------------|
| `/trophy` | Run trophy testing workflow |
| `/deps <path>` | Analyze code dependencies |
| `/grammar <lang> <path>` | Generate grammar for unsupported language |
| `/trophy-workflow` | Full tutorial with step-by-step guidance |
| `/orchestrate verify` | Run deps + trophy in sequence |

---

## Next Steps

After all tests pass:

1. **Commit the generated tests** to your repo
2. **Add to CI/CD** - run `pytest tests/integration/` in your pipeline
3. **Maintain specs** - update OpenSpec when requirements change
4. **Re-run /trophy** - after implementation changes

---

## Need Help?

- Run `/trophy-workflow` for detailed step-by-step guidance
- Check `skills/trophy-workflow/SKILL.md` for patterns
- Check `agents/trophy-guide.md` for agent behavior details
