# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Route Handlers Async Functions Decorated Http

These rules are ALWAYS ACTIVE for all adapter layer modules that expose HTTP endpoints to external clients, all MCP service modules that define REST API boundaries, any module implementing service-to-service HTTP communication interfaces, and health check and monitoring endpoints for operational visibility.

### Rules

- **R-FASTAPI-001** MUST: Route handlers MUST be async functions decorated with HTTP method decorators that specify the endpoint path and response model.
- **R-FASTAPI-002** MUST: Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router.
- **R-FASTAPI-003** MUST: Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions.
- **R-FASTAPI-004** MUST: Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses.
- **R-FASTAPI-005** MUST: Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations.
- **R-FASTAPI-006** MUST: All request and response contracts MUST be defined as Pydantic BaseModel subclasses with typed fields; validation errors MUST return appropriate HTTP 422 responses.

### Verify

```bash
# Discover the project's dependency manifest and identify the resolved version of the web framework
grep -r "fastapi" pyproject.toml poetry.lock requirements.txt 2>/dev/null | head -20

# Confirm the version matches the lock artifact
if [ -f "poetry.lock" ]; then
  grep -A 5 'name = "fastapi"' poetry.lock | grep version
elif [ -f "requirements.lock" ]; then
  grep "fastapi==" requirements.lock
fi

# Locate service boundary modules in adapter and MCP service directories
find . -path ./venv -prune -o -type f -name "*.py" -print | xargs grep -l "APIRouter\|@app\." | grep -E "(adapter|mcp|service)" | head -20

# Verify all route definitions use the decorator pattern with typed models
grep -r "@router\.\|@app\." --include="*.py" | grep -E "(get|post|put|delete|patch)" | head -20

# Identify the project's test suite location and verify integration tests exist
find . -path ./venv -prune -o -type d -name "test*" -print

# Verify Pydantic BaseModel usage in request/response contracts
grep -r "class.*BaseModel" --include="*.py" | grep -E "(Request|Response|Model)" | head -20

# Execute integration tests for service endpoints
if [ -f "pytest.ini" ] || [ -f "pyproject.toml" ]; then
  pytest tests/ -v -k "endpoint or route or service" 2>&1 | head -50
fi
```

**Accept when:**
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses
- Integration tests pass for all service endpoints demonstrating request validation, response serialization, and error handling via HTTPException
- The resolved FastAPI version in the lock artifact is confirmed and matches the dependency manifest
- Health check endpoints follow the pattern: return status, timestamp, version, and service-specific metrics as a dictionary

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers in scope MUST be async functions decorated with HTTP method decorators. All request/response contracts MUST use Pydantic BaseModel. Code review MUST block merge if service endpoints do not follow APIRouter decorator pattern. CI pipeline MUST fail if integration tests are missing for new service boundary endpoints.
</enforcement>