# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Optional Fields Use

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain entities exposed through HTTP APIs, data transfer objects crossing service boundaries, configuration models requiring validation, and monitoring/health check response structures.

### Rules

- **R-PYDANTIC-001** SHOULD: Optional fields SHOULD use `typing.Optional` with default values (`None` or `Field` default) to distinguish required from optional parameters.

### Verify

```bash
# Count BaseModel subclasses in API boundary files
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model declarations in FastAPI routes
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify `response_model` parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- Optional fields in Pydantic models explicitly use `Optional[T]` type hints with default values

<enforcement>
Clause Code MUST NOT skip or defer verification. All new API boundary models MUST declare optional fields using `typing.Optional` with explicit defaults. Code review and CI checks enforce this rule before merge approval.
</enforcement>