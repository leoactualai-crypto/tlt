# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Services Implement Additional

These rules are ALWAYS ACTIVE for all FastAPI-based HTTP services within the monorepo, including Discord adapter endpoints, TLT service monitoring endpoints, and any new service modules exposing REST APIs or webhooks.

### Rules

- **R-FASTAPI-MAY-001** MAY: Services MAY implement additional monitoring endpoints such as /metrics, /status, or /debug for operational visibility beyond basic health checks.

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
- Additional monitoring endpoints (/metrics, /status, /debug) are implemented where operationally beneficial, with structured response models.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI service endpoints MUST include Pydantic validation, response_model declarations, and structured error handling via HTTPException. Code review MUST verify compliance before merge.
</enforcement>