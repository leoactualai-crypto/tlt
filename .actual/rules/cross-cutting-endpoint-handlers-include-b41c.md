# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Handlers Include

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, as well as new service modules exposing business operations via HTTP.

### Rules

- **R-FASTAPI-001** MAY: Endpoint handlers MAY include docstrings describing the operation for API documentation generation.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic model definitions for requests/responses
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handlers
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify async function usage in specific modules
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, etc.)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses
- Endpoint handlers include docstrings for API documentation (where applicable)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing or generating service boundary definitions in the monorepo/tlt domain.
</enforcement>