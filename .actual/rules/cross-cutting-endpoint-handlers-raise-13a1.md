# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Handlers Raise

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, governing how business logic operations are exposed through FastAPI router-based service boundaries.

### Rules

- **R-FASTAPI-001** SHOULD: Endpoint handlers SHOULD raise HTTPException with appropriate status codes (404, 500, 503) for error conditions rather than returning error responses.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic model definitions for request/response handling
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handler functions
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify async function usage in specific service modules
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, @router.delete, etc.)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses
- Error conditions in handlers raise HTTPException with appropriate status codes (400, 404, 500, 503)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All endpoint handlers must follow the FastAPI router pattern with HTTPException-based error handling.
</enforcement>