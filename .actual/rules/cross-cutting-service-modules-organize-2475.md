# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Service Modules Organize

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, governing how business logic operations are exposed through FastAPI router-based service boundaries.

### Rules

- **R-SVC-001** SHOULD: Service modules SHOULD organize related endpoints within a single router instance representing a cohesive business capability.
- **R-SVC-002** MUST: All service endpoint handlers MUST use FastAPI router decorators (@router.get, @router.post, @router.put, @router.delete, @router.patch).
- **R-SVC-003** MUST: All request and response data structures MUST be defined as Pydantic BaseModel subclasses.
- **R-SVC-004** MUST: All endpoint handlers MUST be async functions.
- **R-SVC-005** MUST: Router decorators MUST specify response_model parameter for structured responses.
- **R-SVC-006** MUST: All error conditions MUST use HTTPException with appropriate status codes (400 for validation, 404 for not found, 500 for internal error, 503 for service unavailable).
- **R-SVC-007** SHOULD: Router instances SHOULD be created at module level with appropriate prefix and tags: `router = APIRouter(prefix='/resource', tags=['Resource'])`.
- **R-SVC-008** MUST: Routers MUST be registered with the main FastAPI application using `app.include_router(router)` during application initialization.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic model definitions for requests/responses
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handlers
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify async function usage in specific service module
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, etc.)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses
- HTTPException is used consistently for all error conditions with appropriate status codes
- Router instances are declared at module level with prefix and tags
- Routers are registered with the main FastAPI application during initialization

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All service boundary definitions MUST conform to the FastAPI router pattern with Pydantic models and async handlers. Violations MUST be caught during code review and CI pipeline checks.
</enforcement>