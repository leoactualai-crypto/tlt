# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Http Service Endpoints

These rules are ALWAYS ACTIVE for all FastAPI-based service implementations within the monorepo that expose HTTP endpoints.

### Rules

- **R-FASTAPI-001** MUST: All HTTP service endpoints MUST be defined using FastAPI APIRouter instances with explicit HTTP method decorators (@router.get, @router.post, @router.delete, @router.put, @router.patch).

### Verify

```bash
# Count router decorator usage across service modules
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

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new FastAPI service endpoints must conform to R-FASTAPI-001 before merge.
</enforcement>