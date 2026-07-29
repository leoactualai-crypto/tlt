# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Validation Models Include

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain entities exposed through HTTP APIs, data transfer objects crossing service boundaries, configuration models requiring validation, and monitoring/health check response structures.

### Rules

- **R-PYDANTIC-001** SHOULD: Validation models SHOULD include descriptive field names and Field descriptions to generate self-documenting API contracts.

### Verify

```bash
# Count BaseModel subclasses in API boundary files
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model usage in adapter and service modules
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify response_model parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- Field() constraints are applied for numeric bounds (ge, le) and optional fields (default=None)
- Datetime fields use standard library datetime with Pydantic's automatic ISO 8601 parsing

<enforcement>
Clause Code MUST NOT skip or defer verification. All new API endpoints MUST include Pydantic BaseModel validation models with response_model parameters. Violations require PR comments requesting model addition before merge approval. Repeated violations trigger architecture review escalation.
</enforcement>