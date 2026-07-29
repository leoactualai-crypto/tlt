# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Services Use Optional

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters (discord_adapter) and services (tlt_service), public API contracts exposed for external integration, health check and monitoring endpoints, and CRUD operations on domain entities.

### Rules

- **R-FASTAPI-001** MUST: All HTTP endpoints use FastAPI router decorators with explicit `response_model` declarations.
- **R-FASTAPI-002** MUST: All request bodies are validated using Pydantic BaseModel subclasses with type annotations.
- **R-FASTAPI-003** MUST: Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries.
- **R-FASTAPI-004** MAY: Services MAY use Optional fields with default values in request models to support backward compatibility.

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
- Pydantic models are created in models.py or schemas.py modules adjacent to router definitions
- Field descriptions are included using Field() with description parameter for automatic API documentation
- Separate Create/Update models are used for endpoints that modify state rather than reusing response models

<enforcement>
Claude Code MUST NOT skip or defer verification. All HTTP endpoints in scope MUST comply with R-FASTAPI-001, R-FASTAPI-002, and R-FASTAPI-003. Violations detected by CI pipeline grep checks or code review MUST be remediated before merge. Security testing failures on unvalidated endpoints trigger immediate remediation. Exception requests require security team review and documentation with expiration dates.
</enforcement>