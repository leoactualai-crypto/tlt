# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Http Service Boundaries Defined Fastapi Apirouter

These rules are ALWAYS ACTIVE for all HTTP service boundary modules in adapter layers and MCP services that expose endpoints to external clients or define service-to-service communication interfaces.

### Rules

- **R-HTTP-SB-001** MUST: All HTTP service boundaries MUST be defined using FastAPI APIRouter instances with decorator-based route registration.
- **R-HTTP-SB-002** MUST: Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router.
- **R-HTTP-SB-003** MUST: Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions.
- **R-HTTP-SB-004** MUST: Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses.
- **R-HTTP-SB-005** MUST: Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations.
- **R-HTTP-SB-006** MUST: All request and response contracts MUST be defined as Pydantic BaseModel subclasses with typed fields; validation errors MUST return appropriate HTTP 422 responses.

### Verify

```bash
# Discover the project's dependency manifest and identify the resolved version of the web framework
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "Pipfile" -o -name "poetry.lock" | head -1

# Locate service boundary modules in adapter and MCP service directories
find . -path "*/adapter*" -name "*.py" -o -path "*/mcp*" -name "*.py" | grep -E "(router|route|endpoint)"

# Verify all route definitions use the decorator pattern with typed models
grep -r "@.*\.get\|@.*\.post\|@.*\.put\|@.*\.delete" --include="*.py" | grep -E "response_model|status_code"

# Identify the project's test suite location and verify integration tests exist
find . -path "*/test*" -name "*.py" | grep -E "(service|boundary|endpoint)" | head -5

# Verify APIRouter imports and instantiation
grep -r "from fastapi import APIRouter" --include="*.py"
grep -r "APIRouter()" --include="*.py"

# Verify Pydantic BaseModel usage for contracts
grep -r "from pydantic import BaseModel" --include="*.py"
grep -r "class.*BaseModel" --include="*.py"
```

**Accept when:**
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- Integration tests pass for all service endpoints demonstrating request validation, response serialization, and error handling via HTTPException.
- Health check endpoints return status, timestamp, version, and service-specific metrics as dictionaries using async def.
- Router composition uses include_router with appropriate prefixes for namespace organization.

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if service endpoints do not follow APIRouter decorator pattern. CI pipeline MUST fail if integration tests are missing for new service boundary endpoints. Architecture review is REQUIRED for any service boundary implementation using alternative frameworks or patterns.
</enforcement>