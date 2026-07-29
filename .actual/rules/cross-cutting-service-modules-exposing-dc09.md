# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Service Modules Exposing

These rules are ALWAYS ACTIVE for all FastAPI-based service implementations within the monorepo that expose HTTP endpoints for external clients and monitoring systems.

### Rules

- **R-SVC-001** MUST: All service modules exposing HTTP endpoints MUST implement a `/health` endpoint returning service status, timestamp, and operational metrics.
- **R-SVC-002** MUST: All endpoint handlers that accept structured input or return structured output MUST use Pydantic BaseModel subclasses for validation.
- **R-SVC-003** MUST: All service modules with HTTP endpoints MUST contain at least one FastAPI APIRouter instance with decorated endpoint methods (@router.get, @router.post, @router.delete, @router.put, @router.patch).
- **R-SVC-004** MUST: All endpoints returning structured responses MUST declare a `response_model` parameter in the route decorator.
- **R-SVC-005** MUST: All error conditions MUST raise HTTPException with appropriate status codes and structured error responses.
- **R-SVC-006** SHOULD: Pydantic model naming conventions SHOULD follow: request models suffixed with 'Create', 'Update', or 'Request'; response models suffixed with 'Response'.
- **R-SVC-007** SHOULD: All endpoints SHOULD be documented with docstrings that appear in OpenAPI schema, including parameter descriptions, example requests/responses, and error conditions.

### Verify

```bash
# Count FastAPI router decorators across service modules
grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions with naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count response_model declarations
grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l

# Count HTTPException usage for error handling
grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l

# Verify /health endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a `/health` endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.
- All new endpoints include response_model declarations and structured error handling via HTTPException.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Pre-commit hooks, CI pipeline static analysis, and code review checklists are mandatory enforcement mechanisms. Violations block merge requests and trigger remediation tickets.
</enforcement>