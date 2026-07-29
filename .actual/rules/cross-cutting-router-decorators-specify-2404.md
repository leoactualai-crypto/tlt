# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Router Decorators Specify

These rules are ALWAYS ACTIVE for all HTTP service endpoints in monorepo/tlt/adapters and monorepo/tlt/services, governing how business logic operations are exposed through FastAPI router-based interfaces.

### Rules

- **R-ROUTER-001** MUST: Router decorators MUST specify response_model parameter when returning structured data to enforce response contract validation.

### Verify

```bash
# Count router decorator usage across service modules
grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions for request/response types
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count async endpoint handlers
grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l

# Verify async function usage in service modules
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py
```

**Accept when:**
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, @router.put, @router.delete, @router.patch)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses
- HTTPException is used consistently for all error conditions with appropriate status codes

<enforcement>
Claude Code MUST NOT skip or defer verification. All service boundary definitions MUST comply with R-ROUTER-001 before merge. Violations block CI pipeline and require architecture review board approval for exceptions.
</enforcement>