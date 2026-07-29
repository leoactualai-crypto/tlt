# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoints Specify Response

These rules are ALWAYS ACTIVE for all FastAPI-based service implementations within the monorepo that expose HTTP endpoints.

### Rules

- **R-FASTAPI-001** MUST: All endpoints MUST specify `response_model` parameter in the router decorator when returning structured data.

### Verify

```bash
# Count router decorators across service modules
grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l

# Count Pydantic BaseModel definitions with naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l

# Count response_model usage in endpoints
grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l

# Count HTTPException usage for error handling
grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l

# Verify health check endpoints exist
grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'
```

**Accept when:**
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a `/health` endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.
- All endpoints in scope specify `response_model` parameter in their router decorator definitions.

<enforcement>
Claude Code MUST NOT skip or defer verification of response_model declarations on all FastAPI endpoints. Violations block merge and require remediation or approved exception.
</enforcement>