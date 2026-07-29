# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Service Boundary Definitions

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, governing how business logic operations are exposed through FastAPI router-based service boundaries.

### Rules

- **R-SBD-001** MUST: All service boundary definitions MUST use FastAPI APIRouter instances to declare HTTP endpoints.
- **R-SBD-002** MUST: All request and response data structures MUST be defined as Pydantic BaseModel subclasses.
- **R-SBD-003** MUST: All endpoint handlers MUST be async functions.
- **R-SBD-004** MUST: Router decorators MUST specify response_model parameter for structured responses.
- **R-SBD-005** MUST: All error conditions MUST use HTTPException with appropriate status codes (400 for validation, 404 for not found, 500 for internal error, 503 for service unavailable).
- **R-SBD-006** SHOULD: Router instances SHOULD be created at module level with appropriate prefix and tags: `router = APIRouter(prefix='/resource', tags=['Resource'])`.
- **R-SBD-007** SHOULD: Pydantic models for requests and responses SHOULD be defined before endpoint handlers with type hints for all fields.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions for request/response types
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
- HTTPException is used consistently for all error conditions with appropriate status codes

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All service boundary definitions must be checked against R-SBD-001 through R-SBD-007 before code review approval.
</enforcement>