# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Handlers Async

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, including health check, monitoring, and administrative endpoints exposed via FastAPI routers.

### Rules

- **R-FASTAPI-001** MUST: All endpoint handlers MUST be async functions to support concurrent request processing.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic model definitions for request/response handling
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handler definitions
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify specific module uses async handlers
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, @router.put, @router.delete, @router.patch)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions (verified via AST inspection or grep patterns)
- Router decorators specify response_model parameter for structured responses
- No synchronous endpoint handlers exist in service boundary definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All endpoint handlers in scope MUST be async functions. Violations block merge and require architecture review board approval for exceptions.
</enforcement>