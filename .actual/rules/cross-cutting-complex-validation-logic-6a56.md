# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Complex Validation Logic

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters and services that expose public API contracts for external integration, health checks, monitoring, and CRUD operations on domain entities.

### Rules

- **R-VALIDATION-001** SHOULD: Complex validation logic (e.g., rating ranges, datetime constraints) SHOULD be enforced through Pydantic field validators or constraints.

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
- Complex validation constraints are declared via Pydantic Field() validators or custom validator methods

<enforcement>
Clause Code MUST NOT skip or defer verification. All new endpoints and modifications to existing endpoints must satisfy the accept criteria before merge.
</enforcement>