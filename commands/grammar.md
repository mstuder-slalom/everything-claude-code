---
description: Generate tree-sitter grammar from source code samples. Use when no existing grammar supports a language (e.g., AS/400 RPG, COBOL, proprietary DSLs).
---

# Grammar Command

This command uses the **tree-sitter MCP server** to generate a custom grammar for unsupported languages.

## What This Command Does

1. **Analyze Sample Files** - Identify patterns, keywords, structures
2. **Generate Grammar** - Create grammar.js for tree-sitter
3. **Build Parser** - Compile the tree-sitter parser
4. **Validate** - Parse sample files to verify accuracy
5. **Register** - Add grammar to registry for use with `/deps`

## When to Use

Use `/grammar` when:
- Working with legacy code (AS/400 RPG, COBOL, CL)
- Analyzing proprietary DSLs
- Tree-sitter doesn't have a grammar for your language
- You need dependency extraction for custom languages

## How It Works

The command calls the tree-sitter MCP server with the `generate-grammar` tool:

```json
{
  "language_name": "rpgle",
  "sample_files": ["/path/to/file1.rpgle", "/path/to/file2.rpgle"],
  "output_dir": "~/.claude/grammars/tree-sitter-rpgle"
}
```

## Example Usage

```
User: /grammar rpgle src/as400/

Agent:
# Grammar Generation: RPGLE

## Analyzing Sample Files

Found 12 RPGLE source files in src/as400/

## Pattern Analysis

**Keywords detected:** 47
- H, F, D, C, P (spec types)
- IF, ELSE, ENDIF, DOW, ENDDO
- CALL, CALLP, RETURN
- CHAIN, READ, WRITE, UPDATE

**Dependency constructs:**
- CALL statements → External program calls
- /COPY directives → Include files
- CHAIN operations → File dependencies

**Comment styles:**
- * in column 7 (fixed format)
- // style (free format)

## Generating Grammar

Created: ~/.claude/grammars/tree-sitter-rpgle/grammar.js

Key rules:
- source_file: Program structure
- h_spec, f_spec, d_spec, c_spec, p_spec: Spec types
- call_statement: CALL/CALLP for dependency tracking
- copy_directive: /COPY for includes
- chain_operation: File access

## Building Parser

Running tree-sitter generate...
Running tree-sitter build...
✅ Parser built successfully

## Validation

Parsed 12 sample files:
✅ 12 files parsed without errors
✅ 45 CALL statements extracted
✅ 8 /COPY directives found
✅ 23 CHAIN operations identified

## Registration

Grammar registered at: ~/.claude/grammars/tree-sitter-rpgle
Extensions: .rpgle, .sqlrpgle

You can now use:
/deps src/as400/
```

## Output Structure

Generated grammars are stored in:

```
~/.claude/grammars/
├── tree-sitter-rpgle/
│   ├── grammar.js              # Grammar rules
│   ├── package.json            # npm package config
│   ├── src/
│   │   └── parser.c            # Compiled parser
│   └── queries/
│       └── dependencies.scm    # Dependency extraction queries
├── tree-sitter-cobol/
│   └── ...
└── registry.json               # Maps extensions to grammars
```

## Dependency Queries

Each grammar includes `queries/dependencies.scm` for extracting deps:

```scheme
; tree-sitter-rpgle/queries/dependencies.scm

; CALL statements
(call_statement
  program_name: (identifier) @dependency.call)

; /COPY directives
(copy_directive
  copy_target: (path) @dependency.include)

; CHAIN operations (file dependencies)
(chain_operation
  file_name: (identifier) @dependency.file)

; External procedure references
(extpgm_reference
  program: (identifier) @dependency.external)
```

## Supported Patterns

The grammar generator can detect:

| Pattern Type | Examples |
|-------------|----------|
| Keywords | IF, ELSE, CALL, RETURN |
| Operators | +, -, *, /, =, <, > |
| Statement types | Declarations, operations, control flow |
| Comment styles | //, /* */, * in column |
| String delimiters | 'single', "double" |
| Dependency constructs | CALL, COPY, INCLUDE, IMPORT |

## Language Examples

### AS/400 RPGLE

```
/grammar rpgle src/rpgle/
```

Detects: H/F/D/C/P specs, CALL, /COPY, CHAIN, free-format statements

### COBOL

```
/grammar cobol src/cobol/
```

Detects: DIVISION/SECTION/PARAGRAPH, CALL, COPY, PERFORM

### CL (Control Language)

```
/grammar cl src/cl/
```

Detects: PGM/ENDPGM, DCL, CALL, CHGVAR

### Custom DSL

```
/grammar mydsl config/
```

Analyzes any structured text to generate grammar

## Limitations

- Grammar generation is heuristic-based
- Complex syntax may require manual refinement
- Generated grammars focus on dependency extraction, not full parsing
- Some manual adjustment of grammar.js may be needed

## Manually Refining Grammar

If the generated grammar needs adjustment:

1. Edit `~/.claude/grammars/tree-sitter-{lang}/grammar.js`
2. Run `tree-sitter generate` in that directory
3. Run `tree-sitter build` to compile
4. Test with `tree-sitter parse <sample-file>`

## Integration with /deps

After generating a grammar:

```
# Generate grammar
/grammar rpgle src/as400/

# Now /deps works with RPGLE files
/deps src/as400/
```

The `/deps` command automatically checks:
1. Built-in grammars (Python, TypeScript, Go, Java)
2. Custom grammars in `~/.claude/grammars/`

## Prerequisites

Requires the tree-sitter MCP server:

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

For building parsers locally, you may also need:
- Node.js (for tree-sitter CLI)
- tree-sitter-cli: `npm install -g tree-sitter-cli`
