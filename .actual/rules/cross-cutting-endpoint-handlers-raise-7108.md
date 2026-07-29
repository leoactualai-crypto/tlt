# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Endpoint Handlers Raise

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters (discord_adapter) and services (tlt_service), public API contracts exposed for external integration, health check and monitoring endpoints, and CRUD operations on domain entities.

### Rules

- **R-FASTAPI-001** MUST: Endpoint handlers MUST raise HTTPException with appropriate status codes (404, 500, 503) for error conditions rather than returning error objects.

### Verify

```bash
# Check for endpoints without response_model declarations
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | wc -l

# Count HTTPException raises
grep -r 'raise HTTPException' --include='*.py' monorepo/tlt/ | wc -l
```

**Accept when:**
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries
- No endpoints return error objects; all error handling uses HTTPException with proper HTTP semantics

<enforcement>
Claude Code MUST NOT skip or defer verification. All endpoints must raise HTTPException for error conditions; returning error objects is a violation.
</enforcement>