# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Http Service Endpoints

These rules are ALWAYS ACTIVE for all HTTP service endpoints in adapters and services that expose endpoints for external integration and monitoring.

### Rules

- **R-HTTP-001** MUST: All HTTP service endpoints MUST be defined using FastAPI APIRouter instances with explicit route decorators (@router.post, @router.get, @router.delete, @router.put, @router.patch).
- **R-HTTP-002** MUST: All request bodies MUST be validated using Pydantic BaseModel subclasses with explicit type annotations.
- **R-HTTP-003** MUST: All endpoint handlers MUST include explicit response_model declarations in route decorators.
- **R-HTTP-004** MUST: Error conditions MUST raise HTTPException with appropriate status codes rather than returning error dictionaries.
- **R-HTTP-005** SHOULD: Pydantic models SHOULD be created in a models.py or schemas.py module adjacent to router definitions for discoverability.
- **R-HTTP-006** SHOULD: For endpoints that modify state, SHOULD use separate Create/Update models rather than reusing response models to enforce immutability.
- **R-HTTP-007** SHOULD: Pydantic models SHOULD include Field() with description parameter for automatic API documentation.

### Verify

```bash
# Check for endpoints without response_model declarations
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' --include='*.py' | wc -l

# Count HTTPException usage
grep -r 'raise HTTPException' --include='*.py' | wc -l

# Verify type annotations on endpoint handlers
mypy --strict --include-untyped-defs .
```

**Accept when:**
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries
- Static analysis with mypy verifies type annotations on endpoint handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All new HTTP endpoints MUST comply with R-HTTP-001 through R-HTTP-004 before merge. Code review MUST block merge if Pydantic validation is missing for request bodies. CI pipeline MUST fail if endpoints lack response_model declarations.
</enforcement>