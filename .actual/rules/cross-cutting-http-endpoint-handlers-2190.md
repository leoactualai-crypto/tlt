# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Http Endpoint Handlers

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, as well as new service modules exposing business operations via HTTP.

### Rules

- **R-FASTAPI-001** MUST: HTTP endpoint handlers MUST be decorated with router method decorators (@router.post, @router.get, @router.delete, @router.put, @router.patch) specifying the URL path.
- **R-FASTAPI-002** MUST: All request and response data structures MUST be defined as Pydantic BaseModel subclasses.
- **R-FASTAPI-003** MUST: All endpoint handlers MUST be async functions.
- **R-FASTAPI-004** MUST: Router decorators MUST specify the response_model parameter for structured responses.
- **R-FASTAPI-005** MUST: HTTPException MUST be used for all error conditions with appropriate status codes (400 for validation, 404 for not found, 500 for internal error, 503 for service unavailable).
- **R-FASTAPI-006** MUST: Router instances MUST be created at module level with appropriate prefix and tags: `router = APIRouter(prefix='/resource', tags=['Resource'])`.
- **R-FASTAPI-007** MUST: Routers MUST be registered with the main FastAPI application using `app.include_router(router)` in application initialization.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic model definitions for request/response handling
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handlers
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify async function usage in specific service module
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, @router.put, @router.delete, @router.patch)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses
- HTTPException is used consistently for error handling with appropriate status codes
- Router instances are declared at module level with prefix and tags
- Routers are registered with the main FastAPI application during initialization

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Violations block merge and require architecture review board approval for exceptions.
</enforcement>