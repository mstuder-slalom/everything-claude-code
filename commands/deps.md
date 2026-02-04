---
description: Parse source code dependencies using tree-sitter. Confirms linkages and validates code map accuracy.
---

# Deps Command

This command uses the **tree-sitter MCP server** to parse and analyze code dependencies.

## What This Command Does

1. **Detect Language** - Identify programming language from files
2. **Parse Source Files** - Use tree-sitter grammar to parse code
3. **Extract Dependencies** - Find imports, exports, function calls
4. **Build Dependency Graph** - Map file and function level deps
5. **Validate Code Map** - Compare against CODEMAP.md
6. **Report Discrepancies** - Show missing or incorrect dependencies

## When to Use

Use `/deps` when:
- Understanding unfamiliar codebase structure
- Validating a CODEMAP.md against actual code
- Analyzing dependency chains before refactoring
- Preparing for trophy testing (provides context)
- Checking what a file imports/exports

## How It Works

The command calls the tree-sitter MCP server with the `tree-sitter-deps` tool:

```json
{
  "path": "/path/to/analyze",
  "language": "python",
  "depth": "full"
}
```

## Example Usage

```
User: /deps src/

Agent:
# Dependency Analysis: src/

## Language Detected: Python

## Files Analyzed: 6

## File Dependencies

### processor.py
**Imports:**
- External: fitz, subprocess
- Internal: llm_client, utils

**Exports:**
- Functions: process_document, detect_type
- Classes: DocumentProcessor

### coordinator.py
**Imports:**
- Internal: processor, exporter

**Exports:**
- Functions: run_pipeline, batch_process

## Dependency Graph

### File Level
```
coordinator.py
├── processor.py
│   ├── llm_client.py
│   └── utils.py
└── exporter.py
    └── utils.py
```

### Function Level
```
coordinator.run_pipeline
├── processor.process_document
│   ├── llm_client.analyze
│   └── utils.format_output
└── exporter.export_to_excel
```

## Code Map Validation

Comparing against CODEMAP.md...

✅ All documented dependencies confirmed
⚠️ Undocumented dependency: processor.py → subprocess (external)
```

## Output Format

The tree-sitter-deps tool returns structured JSON:

```json
{
  "analysis": {
    "path": "/path/to/src",
    "language": "python",
    "files_analyzed": 6
  },
  "files": {
    "processor.py": {
      "imports": {
        "external": ["fitz", "subprocess"],
        "internal": ["llm_client", "utils"]
      },
      "exports": {
        "functions": ["process_document", "detect_type"],
        "classes": ["DocumentProcessor"],
        "constants": []
      },
      "functions": {
        "process_document": {
          "calls": [
            {"target": "detect_type", "line": 45},
            {"target": "llm_client.analyze", "line": 78}
          ],
          "params": ["file_path", "config"],
          "returns": "dict"
        }
      }
    }
  },
  "dependency_graph": {
    "file_level": {
      "coordinator.py": ["processor.py", "exporter.py"]
    },
    "function_level": {
      "coordinator.run_pipeline": [
        "processor.process_document",
        "exporter.export_to_excel"
      ]
    }
  }
}
```

## Supported Languages

| Language | Extensions | Grammar |
|----------|-----------|---------|
| Python | `.py` | tree-sitter-python |
| TypeScript | `.ts`, `.tsx` | tree-sitter-typescript |
| JavaScript | `.js`, `.jsx` | tree-sitter-javascript |
| Go | `.go` | tree-sitter-go |
| Java | `.java` | tree-sitter-java |

For unsupported languages, use `/grammar` to generate a custom grammar.

## Analysis Depth Options

- **file** - Imports and exports only
- **function** - Add function-level call graph
- **full** - Complete analysis with all details

## Integration with Other Commands

- Use `/deps` before `/trophy` to understand code structure
- Use `/deps` to validate CODEMAP.md accuracy
- Use `/deps` during `/orchestrate` workflows
- Use `/grammar` if language grammar is missing

## Prerequisites

Requires the tree-sitter MCP server to be configured:

```json
// .claude/settings.json
{
  "mcpServers": {
    "tree-sitter": {
      "command": "uvx",
      "args": ["mcp-tree-sitter"]
    }
  }
}
```

## Custom Grammars

For languages not supported out-of-the-box (AS/400 RPG, COBOL, etc.), first generate a grammar:

```
/grammar rpgle src/as400/
```

Then `/deps` will work with the custom grammar.
