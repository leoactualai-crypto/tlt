# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Pydantic Validation Models

These rules are ALWAYS ACTIVE for all HTTP endpoints in adapters (discord_adapter) and services (tlt_service), public API contracts exposed for external integration, health check and monitoring endpoints, and CRUD operations on domain entities.

### Rules

- **R-PYDANTIC-001** MUST: All Pydantic validation models MUST use descriptive class names ending in Create, Update, Response, or Request to indicate their role in the request/response cycle.

### Verify

```bash
# Check for endpoints without response_model declarations
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'

# Count Pydantic BaseModel subclasses
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | wc -l

# Count HTTPException usage for error handling
grep -r 'raise HTTPException' --include='*.py' monorepo/tlt/ | wc -l
```

**Accept when:**
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries
- All Pydantic validation model class names follow the Create, Update, Response, or Request naming convention

<enforcement>
Claude Code MUST NOT skip or defer verification. All endpoints must be checked for response_model declarations, and all Pydantic models must follow the naming convention before code is approved.
</enforcement>