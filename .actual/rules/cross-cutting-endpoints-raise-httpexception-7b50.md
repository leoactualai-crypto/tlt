# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoints Raise Httpexception

These rules are ALWAYS ACTIVE for all FastAPI-based HTTP services within the monorepo, including Discord adapter endpoints, TLT service monitoring endpoints, and any new service modules exposing REST APIs or webhooks.

### Rules

- **R-FASTAPI-001** MUST: All endpoints MUST raise HTTPException with appropriate status codes (404, 500, 503) for error conditions rather than returning unstructured error responses.

### Verify

```bash
# Count FastAPI router decorators in use
grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions with standard naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count response_model declarations
grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l

# Count HTTPException raise statements
grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l

# Verify health check endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a /health endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.
- All endpoints raising HTTPException use appropriate status codes (404, 500, 503) for error conditions.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints MUST comply with R-FASTAPI-001 before merge. CI pipeline MUST fail if endpoints are added without proper HTTPException error handling.
</enforcement>