# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Services Implement Structured

These rules are ALWAYS ACTIVE for all FastAPI-based HTTP services within the monorepo, including Discord adapter endpoints, TLT service monitoring endpoints, and any new service modules exposing REST APIs or webhooks.

### Rules

- **R-FASTAPI-001** SHOULD: Services SHOULD implement structured logging using logger instances (logging.getLogger(__name__) or loguru.logger) within endpoint handlers for observability.
- **R-FASTAPI-002** MUST: All service modules with HTTP endpoints MUST contain at least one FastAPI APIRouter instance with decorated endpoint methods (@router.get, @router.post, @router.delete, @router.put, @router.patch).
- **R-FASTAPI-003** MUST: All endpoint handlers that accept structured input or return structured output MUST use Pydantic BaseModel subclasses for validation.
- **R-FASTAPI-004** MUST: All services exposing HTTP endpoints MUST implement a /health endpoint returning structured status information.
- **R-FASTAPI-005** MUST: All endpoints accepting or returning structured data MUST declare response_model in the route decorator.
- **R-FASTAPI-006** MUST: Error handling MUST use HTTPException for consistent error responses across services.

### Verify

```bash
# Count FastAPI router decorators across service modules
grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions with standard naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count response_model declarations
grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l

# Count HTTPException usage for error handling
grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l

# Verify health check endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a /health endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.
- All new endpoints include response_model declarations and HTTPException error handling.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, code review checklists, and automated OpenAPI schema validation are mandatory. Violations block merge requests and CI pipeline execution.
</enforcement>