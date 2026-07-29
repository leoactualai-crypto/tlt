# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Endpoint Response Models

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters (discord_adapter) and services (tlt_service), public API contracts exposed for external integration, health check and monitoring endpoints, and CRUD operations on domain entities.

### Rules

- **R-FASTAPI-001** MUST: All endpoint response models MUST be declared using the `response_model` parameter in route decorators.
- **R-FASTAPI-002** MUST: All request bodies MUST be validated using Pydantic BaseModel subclasses with type annotations.
- **R-FASTAPI-003** MUST: Error conditions MUST raise HTTPException with appropriate status codes rather than returning error dictionaries.
- **R-FASTAPI-004** SHOULD: Create Pydantic models in a models.py or schemas.py module adjacent to router definitions for discoverability.
- **R-FASTAPI-005** SHOULD: Use descriptive field names and include Field() with description parameter for automatic API documentation.
- **R-FASTAPI-006** SHOULD: For endpoints that modify state, use separate Create/Update models rather than reusing response models to enforce immutability.
- **R-FASTAPI-007** SHOULD: Include example values in Pydantic models using Config.schema_extra for better OpenAPI documentation and testing.

### Verify

```bash
# Check for endpoints without response_model declarations
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | wc -l

# Count HTTPException usage
grep -r 'raise HTTPException' --include='*.py' monorepo/tlt/ | wc -l
```

**Accept when:**
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries
- Static analysis with mypy verifies type annotations on endpoint handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All new endpoints and modifications to existing endpoints must pass the verify commands before acceptance.
</enforcement>