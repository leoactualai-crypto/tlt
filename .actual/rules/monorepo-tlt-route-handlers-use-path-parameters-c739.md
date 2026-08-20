# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Route Handlers Use Path Parameters Query

These rules are ALWAYS ACTIVE for all adapter layer modules that expose HTTP endpoints to external clients, all MCP service modules that define REST API boundaries, any module implementing service-to-service HTTP communication interfaces, and health check and monitoring endpoints for operational visibility.

### Rules

- **R-FASTAPI-001** MAY: Route handlers MAY use path parameters, query parameters, and request body parameters with type annotations for automatic validation.
- **R-FASTAPI-002** MUST: Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router.
- **R-FASTAPI-003** SHOULD: Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions.
- **R-FASTAPI-004** MUST: Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses.
- **R-FASTAPI-005** MUST: Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations.
- **R-FASTAPI-006** MUST: All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- **R-FASTAPI-007** MUST: All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- **R-FASTAPI-008** MUST: Establish HTTPException usage conventions with standard status codes; implement centralized exception handlers for common error types; document error response schemas.

### Verify

```bash
# Discover the project's dependency manifest and identify the resolved version of the web framework
grep -r "fastapi" pyproject.toml poetry.lock requirements.txt 2>/dev/null | head -20

# Confirm the version matches the lock artifact
if [ -f "poetry.lock" ]; then
  grep -A 5 'name = "fastapi"' poetry.lock | grep version
elif [ -f "requirements.txt" ]; then
  grep fastapi requirements.txt
fi

# Locate service boundary modules in adapter and MCP service directories
find . -path ./venv -prune -o -type f -name "*.py" -print | xargs grep -l "APIRouter" | grep -E "(adapter|mcp|service)" | head -20

# Verify all route definitions use the decorator pattern with typed models
grep -r "@.*\.get\|@.*\.post\|@.*\.put\|@.*\.delete" --include="*.py" | grep -E "(adapter|mcp|service)" | head -20

# Identify the project's test suite location
find . -path ./venv -prune -o -type d -name "test*" -o -type d -name "*test" -print | head -10

# Execute integration tests for service endpoints
if [ -f "pytest.ini" ] || [ -f "pyproject.toml" ]; then
  python -m pytest -v --tb=short -k "test_" 2>&1 | head -50
fi

# Verify Pydantic BaseModel usage in contracts
grep -r "from pydantic import\|BaseModel" --include="*.py" | grep -E "(adapter|mcp|service)" | head -20

# Check for HTTPException usage patterns
grep -r "HTTPException" --include="*.py" | grep -E "(adapter|mcp|service)" | head -20
```

**Accept when:**
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- Integration tests pass for all service endpoints demonstrating request validation, response serialization, and error handling via HTTPException.
- The project's dependency manifest resolves to a FastAPI version with confirmed APIRouter and Pydantic integration support.
- Health check endpoints return status, timestamp, version, and service-specific metrics as dictionaries using async def.
- Centralized exception handlers are implemented for common error types with documented error response schemas.

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if service endpoints do not follow APIRouter decorator pattern. CI pipeline MUST fail if integration tests are missing for new service boundary endpoints. Architecture review is REQUIRED for any service boundary implementation using alternative frameworks or patterns.
</enforcement>