# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Fastapi Router Endpoints

These rules are ALWAYS ACTIVE for all FastAPI router endpoints and domain model definitions exposed through HTTP API boundaries within the codebase.

### Rules

- **R-PYDANTIC-001** MUST: FastAPI router endpoints MUST declare `response_model` parameter using Pydantic BaseModel subclasses for automatic response validation and OpenAPI schema generation.
- **R-PYDANTIC-002** MUST: All request/response models crossing API boundaries MUST inherit from `pydantic.BaseModel` and define Field-level constraints (ge, le, Optional, etc.) to enforce domain invariants.
- **R-PYDANTIC-003** SHOULD: Use `Field()` with descriptive constraints for numeric bounds: `Field(ge=0.0, le=1.0, description='...')` and `Field(default=None)` for optional fields.
- **R-PYDANTIC-004** SHOULD: For datetime fields, import from standard library; Pydantic handles ISO 8601 string parsing automatically.
- **R-PYDANTIC-005** MAY: Legacy endpoints may request exception EXC-001 for gradual migration from dict-based validation with documented timeline.
- **R-PYDANTIC-006** MAY: Performance-critical paths demonstrating measurable Pydantic overhead (>10ms p99 latency) may request exception EXC-002 with benchmark data.

### Verify

```bash
# Count BaseModel subclasses across adapter and service modules
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
- Pydantic import verification succeeds without errors

<enforcement>
Clause Code MUST NOT skip or defer verification. All new API endpoints require Pydantic BaseModel validation before merge approval. Violations trigger PR comments requesting model addition and CI failures on unvalidated dictionary parameters in FastAPI routes.
</enforcement>