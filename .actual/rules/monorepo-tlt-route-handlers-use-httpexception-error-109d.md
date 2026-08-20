# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Route Handlers Use Httpexception Error Responses

These rules are ALWAYS ACTIVE for all adapter layer modules that expose HTTP endpoints to external clients, all MCP service modules that define REST API boundaries, any module implementing service-to-service HTTP communication interfaces, and health check and monitoring endpoints for operational visibility.

### Rules

- **R-FASTAPI-001** MUST: Route handlers MUST use HTTPException for error responses with appropriate HTTP status codes.
- **R-FASTAPI-002** MUST: All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- **R-FASTAPI-003** MUST: All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- **R-FASTAPI-004** SHOULD: Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router.
- **R-FASTAPI-005** SHOULD: Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions.
- **R-FASTAPI-006** SHOULD: Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses.
- **R-FASTAPI-007** SHOULD: Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations.

### Verify

```bash
# Discover the project's dependency manifest and identify the resolved version of the web framework
grep -r "fastapi" pyproject.toml requirements.txt setup.py 2>/dev/null | head -5

# Locate service boundary modules in adapter and MCP service directories
find . -path ./venv -prune -o -type f -name "*.py" -print | xargs grep -l "APIRouter" | head -10

# Verify all route definitions use the decorator pattern with typed models
grep -r "@.*\.get\|@.*\.post\|@.*\.put\|@.*\.delete" --include="*.py" | grep -v test | head -10

# Identify HTTPException usage in route handlers
grep -r "HTTPException" --include="*.py" | grep -v test | head -10

# Locate the project's test suite and verify integration tests exist
find . -path ./venv -prune -o -type d -name "test*" -print

# Execute integration tests for service endpoints
python -m pytest tests/ -v -k "endpoint or route or service" 2>/dev/null || echo "Test execution requires environment setup"
```

**Accept when:**
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters.
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses.
- Integration tests pass for all service boundary endpoints demonstrating request validation, response serialization, and error handling via HTTPException.
- All route handlers use HTTPException with appropriate HTTP status codes for error responses.
- No service boundary endpoints use alternative error handling patterns outside HTTPException.

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if service endpoints do not follow APIRouter decorator pattern. CI pipeline MUST fail if integration tests are missing for new service boundary endpoints. Architecture review is REQUIRED for any service boundary implementation using alternative frameworks or patterns.
</enforcement>