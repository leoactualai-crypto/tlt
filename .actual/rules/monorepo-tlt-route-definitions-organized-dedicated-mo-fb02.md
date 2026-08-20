# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Route Definitions Organized Dedicated Modules Separate

These rules are ALWAYS ACTIVE for all adapter layer modules that expose HTTP endpoints to external clients, all MCP service modules that define REST API boundaries, any module implementing service-to-service HTTP communication interfaces, and health check and monitoring endpoints for operational visibility.

### Rules

- **R-ROUTE-001** SHOULD: Route definitions SHOULD be organized in dedicated route modules separate from business logic implementation.
- **R-ROUTE-002** MUST: All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- **R-ROUTE-003** MUST: All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- **R-ROUTE-004** SHOULD: Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router.
- **R-ROUTE-005** SHOULD: Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions.
- **R-ROUTE-006** SHOULD: Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses.
- **R-ROUTE-007** SHOULD: Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations.

### Verify

```bash
# Discover the project's dependency manifest and identify the resolved version of the web framework
grep -r "fastapi" pyproject.toml poetry.lock requirements.txt 2>/dev/null | head -5

# Locate service boundary modules in adapter and MCP service directories
find . -path ./venv -prune -o -type f -name "*router*.py" -o -name "*routes*.py" | grep -E "(adapter|mcp|service)" | head -20

# Verify all route definitions use the decorator pattern with typed models
grep -r "@.*\.get\|@.*\.post\|@.*\.put\|@.*\.delete" --include="*.py" | grep -E "(adapter|mcp|service)" | head -10

# Identify the project's test suite location and verify integration tests exist
find . -path ./venv -prune -o -type d -name "test*" -o -name "*test" | head -5

# Verify Pydantic BaseModel usage in route modules
grep -r "from pydantic import\|BaseModel" --include="*.py" | grep -E "(adapter|mcp|service|route)" | head -10

# Confirm APIRouter instantiation pattern
grep -r "APIRouter()" --include="*.py" | head -10
```

**Accept when:**
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses
- Integration tests pass for all service endpoints demonstrating request validation, response serialization, and error handling via HTTPException
- The project's dependency manifest resolves to a FastAPI version with confirmed APIRouter and Pydantic integration support
- Health check endpoints follow the pattern of returning status, timestamp, version, and service-specific metrics

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if service endpoints do not follow APIRouter decorator pattern. CI pipeline MUST fail if integration tests are missing for new service boundary endpoints. Architecture review is REQUIRED for any service boundary implementation using alternative frameworks or patterns.
</enforcement>