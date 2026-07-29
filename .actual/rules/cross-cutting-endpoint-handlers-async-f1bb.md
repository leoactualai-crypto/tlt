# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Endpoint Handlers Async

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters (discord_adapter) and services (tlt_service), public API contracts exposed for external integration, health check and monitoring endpoints, and CRUD operations on domain entities.

### Rules

- **R-FASTAPI-001** SHOULD: Endpoint handlers SHOULD be async functions to support concurrent request processing.

### Verify

```bash
# Check for endpoints without response_model declarations
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | wc -l

# Count HTTPException usage for error handling
grep -r 'raise HTTPException' --include='*.py' monorepo/tlt/ | wc -l
```

**Accept when:**
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries
- Endpoint handlers are defined as async functions for concurrent request processing

<enforcement>
Clause Code MUST NOT skip or defer verification. All new HTTP endpoints MUST include response_model declarations, Pydantic validation for request bodies, and HTTPException error handling. Violations block merge; exceptions require security team review and documentation.
</enforcement>