# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Async Endpoint Handlers

These rules are ALWAYS ACTIVE for all FastAPI-based service implementations within the monorepo, including Discord adapter endpoints, TLT service monitoring endpoints, and any new service modules exposing REST APIs or webhooks.

### Rules

- **R-FASTAPI-001** SHOULD: Async endpoint handlers SHOULD be used for I/O-bound operations including external API calls, database queries, and message queue interactions.
- **R-FASTAPI-002** MUST: All service modules with HTTP endpoints MUST contain at least one FastAPI APIRouter instance with decorated endpoint methods (@router.get, @router.post, @router.delete, @router.put, @router.patch).
- **R-FASTAPI-003** MUST: All endpoint handlers that accept structured input or return structured output MUST use Pydantic BaseModel subclasses for validation.
- **R-FASTAPI-004** MUST: All services exposing HTTP endpoints MUST implement a /health endpoint returning structured status information.
- **R-FASTAPI-005** MUST: All endpoints accepting or returning structured data MUST declare response_model in the route decorator.
- **R-FASTAPI-006** MUST: Error handling MUST use HTTPException for consistent error responses across services.
- **R-FASTAPI-007** SHOULD: Pydantic model naming conventions SHOULD follow: request models suffixed with 'Create', 'Update', or 'Request'; response models suffixed with 'Response'.
- **R-FASTAPI-008** SHOULD: All endpoints SHOULD be documented with docstrings that appear in OpenAPI schema, including parameter descriptions, example requests/responses, and error conditions.

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

# Verify health check endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a /health endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.
- All new endpoints include response_model declarations and HTTPException error handling.
- Pydantic models follow established naming conventions (Request/Response/Create/Update suffixes).

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks and CI pipeline static analysis MUST validate router decorator usage, Pydantic model validation, response_model declarations, and HTTPException error handling. Code review MUST verify all new service endpoints comply with these rules before merge. Violations result in CI pipeline failure and code review block.
</enforcement>