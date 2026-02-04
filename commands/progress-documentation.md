# /progress-documentation - Session Progress Documentation

Generate comprehensive progress documentation with tiered detail levels (executive summary, medium, low) for work sessions and implementations.

## Usage

```bash
/progress-documentation                           # Document current session
/progress-documentation "Feature implementation"  # Document specific work
/progress-documentation --output ~/Documents/docs # Custom output location
```

## What It Creates

A markdown file with:

1. **Executive Summary** - Key outcomes, status table, immediate usage (30 sec read)
2. **Medium-Level Detail** - Components built, files changed, installation method (5 min read)
3. **Low-Level Detail** - Architecture, code examples, configs, troubleshooting (deep dive)

## Workflow

### Step 1: Gather Context

Identify from the current session:
- What was accomplished?
- What components were created/modified/deleted?
- What is the current status?
- What commands verify the work?

### Step 2: Determine Output Location

```bash
# Find or create docs directory
mkdir -p ~/Documents/dev_work_docs
```

### Step 3: Generate Filename

Format: `{descriptive-name}_{YYYY-MM-DD}_{HHMM}.md`

```bash
date "+%Y-%m-%d_%H%M"
```

### Step 4: Write Document

Follow the tiered structure:

```markdown
# {Title}

**Date:** {Date} | {Time}
**Project:** {Context}
**Status:** {Complete | In Progress | Blocked}

---

## Executive Summary
{High-level outcomes, status table, quick start commands}

---

## Medium-Level Detail
{What was built, components, files changed, installation steps}

---

## Low-Level Detail
{Architecture, code examples, configs, troubleshooting, verification}

---

## References
{Links to repos, docs, related work}

## Next Steps
{Action items}
```

### Step 5: Confirm Creation

```bash
ls -la {output_path}
wc -l {output_path}
```

## Output Example

**Filename:** `auth-system-implementation_2026-02-03_1430.md`

**Content Preview:**
```markdown
# Auth System Implementation

**Date:** February 3, 2026 | 2:30 PM
**Project:** MyApp Backend
**Status:** Complete

---

## Executive Summary

Implemented JWT-based authentication with refresh tokens...

| Component | Status | Location |
|-----------|--------|----------|
| Auth Service | Complete | `src/services/auth.ts` |
| JWT Middleware | Complete | `src/middleware/jwt.ts` |

### Immediate Usage
\`\`\`bash
npm run dev
curl -X POST http://localhost:3000/api/auth/login
\`\`\`
```

## Best Practices

| Level | Audience | Focus |
|-------|----------|-------|
| Executive | Managers, stakeholders | Outcomes, status, quick start |
| Medium | Team members | What, why, how (overview) |
| Low | Implementers | Full technical detail |

---

*Part of [Everything Claude Code](https://github.com/affaan-m/everything-claude-code)*
