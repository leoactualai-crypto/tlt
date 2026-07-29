# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Request Response

These rules are ALWAYS ACTIVE for all FastAPI-based service implementations within the monorepo that expose HTTP endpoints for external clients and monitoring systems.

### Rules

- **R-ENDPOINT-001** MUST: All endpoint request and response payloads MUST be validated using Pydantic BaseModel subclasses with explicit type annotations.

### Verify

```bash
# Count FastAPI router decorators across service modules
grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions with standard naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count response_model declarations in endpoint handlers
grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l

# Count HTTPException usage for structured error handling
grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l

# Verify health check endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a /health endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.

<enforcement>
Claude Code MUST NOT skip or defer verification of Pydantic BaseModel validation on all FastAPI endpoints. Pre-commit hooks and CI pipeline static analysis MUST fail if new endpoints are added without response_model declarations or Pydantic validation. Code review MUST block merge requests that introduce unvalidated endpoints.
</enforcement>