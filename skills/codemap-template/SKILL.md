---
name: codemap-template
description: Generate and maintain CODEMAP.md files that document project structure, dependencies, and data flow for codebase navigation and AI agent context.
---

# Code Map Template

A structured format for documenting project architecture, dependencies, and data flow. CODEMAP.md files help developers and AI agents quickly understand codebase structure.

## When to Activate

- Creating a new CODEMAP.md for a project
- Updating an existing code map after significant changes
- Validating a code map against actual code structure
- Understanding an unfamiliar codebase

## CODEMAP.md Template

Create a `CODEMAP.md` file at the project root with this structure:

```markdown
# Code Map: {project-name}

## Overview
{1-2 sentence description of what this project does}

## Structure
```
{directory tree showing key files and folders}
```

## Entry Points
- {primary entry point with description}
- {secondary entry points if applicable}

## Key Dependencies
- {source file} → {target file} ({reason/purpose})
- ...

## Data Flow
1. {step 1 in the main data/request flow}
2. {step 2}
...

## External Dependencies
- {package/library}: {what it's used for}
```

## Example: Document Processing Pipeline

```markdown
# Code Map: doc-ingestion-pipeline

## Overview
Python pipeline for automated document processing. Extracts text/visuals from PPTX/PDF/DOCX, generates summaries, exports to Excel.

## Structure
```
doc_ingestion/
├── coordinator.py    # Pipeline orchestration
├── processor.py      # Document extraction
├── exporter.py       # Excel output
├── extractor.py      # IE value extraction
├── llm_client.py     # LLM API wrapper
└── utils.py          # Shared utilities
```

## Entry Points
- `python -m doc_ingestion config.json` - CLI batch processing
- `coordinator.run_pipeline(path)` - Programmatic entry

## Key Dependencies
- coordinator.py → processor.py (calls process_document)
- coordinator.py → exporter.py (calls export_to_excel)
- processor.py → llm_client.py (visual analysis, summaries)
- extractor.py → llm_client.py (IE field extraction)

## Data Flow
1. Coordinator loads config, iterates input files
2. Processor extracts text + visuals per page
3. LLM client analyzes visuals, generates summaries
4. Exporter writes results to Excel
5. Extractor reads Excel, extracts IE fields

## External Dependencies
- PyMuPDF (fitz): PDF parsing and rendering
- python-pptx: PowerPoint extraction
- openpyxl: Excel manipulation
- anthropic: Claude API for LLM calls
```

## Example: Web API Service

```markdown
# Code Map: user-service

## Overview
REST API service for user management with authentication, profile management, and team features.

## Structure
```
src/
├── main.ts           # Application entry
├── routes/
│   ├── auth.ts       # Authentication endpoints
│   ├── users.ts      # User CRUD endpoints
│   └── teams.ts      # Team management
├── services/
│   ├── auth.ts       # Auth logic
│   ├── users.ts      # User business logic
│   └── teams.ts      # Team business logic
├── models/
│   ├── user.ts       # User entity
│   └── team.ts       # Team entity
└── middleware/
    ├── auth.ts       # JWT validation
    └── validation.ts # Request validation
```

## Entry Points
- `npm start` - Start production server
- `npm run dev` - Start development server with hot reload
- `GET /health` - Health check endpoint

## Key Dependencies
- routes/auth.ts → services/auth.ts (login, register, refresh)
- routes/users.ts → services/users.ts (CRUD operations)
- middleware/auth.ts → services/auth.ts (token validation)
- services/*.ts → models/*.ts (data access)

## Data Flow
1. Request hits route handler
2. Middleware validates JWT (if protected)
3. Middleware validates request body
4. Service layer processes business logic
5. Model layer interacts with database
6. Response returned to client

## External Dependencies
- express: HTTP server framework
- jsonwebtoken: JWT generation/validation
- bcrypt: Password hashing
- prisma: Database ORM
- zod: Schema validation
```

## Validation with Tree-sitter

When the `mcp-tree-sitter` MCP server is available, validate your CODEMAP.md against actual code:

1. **Read the CODEMAP.md** to understand documented structure
2. **Run tree-sitter-deps** to get actual dependencies:
   ```json
   {
     "path": "/path/to/project",
     "depth": "full"
   }
   ```
3. **Compare** documented vs actual:
   - Missing files in CODEMAP.md
   - Undocumented dependencies
   - Incorrect data flow descriptions
4. **Update** CODEMAP.md with any discrepancies

## Best Practices

### Keep It Current
- Update CODEMAP.md when adding/removing major files
- Review during PR reviews for architectural changes
- Use tree-sitter validation periodically

### Right Level of Detail
- Include files that define key abstractions
- Skip generated files, tests, config files
- Focus on "what would a new developer need to know"

### Accurate Dependencies
- Only document meaningful dependencies
- Include the reason/purpose for each dependency
- Use file names, not just module names

### Clear Data Flow
- Start from external inputs (HTTP requests, CLI args, file inputs)
- Follow the main path through the system
- End at external outputs (responses, files, database)

## Integration with AI Agents

CODEMAP.md provides essential context for AI agents:

| Agent Use Case | CODEMAP.md Section |
|---------------|-------------------|
| "Where should I add this feature?" | Structure, Entry Points |
| "What does this file depend on?" | Key Dependencies |
| "How does data flow through the system?" | Data Flow |
| "What external libraries are used?" | External Dependencies |
| "What is this project for?" | Overview |

## Generating CODEMAP.md

To generate a CODEMAP.md for a project:

1. **Explore the project structure**
   - Identify the main source directory
   - Find entry points (main.py, index.ts, cmd/main.go)
   - Note the key abstraction layers

2. **Map dependencies**
   - Use `tree-sitter-deps` if available
   - Or manually trace imports/requires

3. **Document data flow**
   - Start from entry points
   - Follow function calls
   - Note where data is transformed

4. **List external dependencies**
   - Check package.json, requirements.txt, go.mod
   - Note what each is used for

## File Placement

Always place CODEMAP.md at the project root, alongside:
- README.md (user documentation)
- CONTRIBUTING.md (contribution guidelines)
- CODEMAP.md (architecture documentation)
