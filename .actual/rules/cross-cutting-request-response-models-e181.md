# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Request Response Models

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain entities exposed through HTTP APIs, data transfer objects crossing service boundaries, configuration models requiring validation, and monitoring/health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: All API request and response models MUST inherit from `pydantic.BaseModel` to enforce runtime type validation and serialization.
- **R-PYDANTIC-002** MUST: Use `Field()` for constraints on numeric bounds (e.g., `Field(ge=0.0, le=1.0, description='...')`) and optional fields (e.g., `Field(default=None)`).
- **R-PYDANTIC-003** MUST: Declare `response_model` parameter in FastAPI decorators (e.g., `@router.post('/', response_model=YourModel)`) for automatic validation and serialization.
- **R-PYDANTIC-004** SHOULD: Import `BaseModel` from `pydantic` and define request/response classes before router endpoint definitions.
- **R-PYDANTIC-005** SHOULD: For datetime fields, import from standard library; Pydantic handles ISO 8601 string parsing automatically.
- **R-PYDANTIC-006** MAY: Use `model_validate()` or `model_validate_json()` for performance optimization in high-throughput endpoints (>1000 req/s) after profiling demonstrates measurable overhead.

### Verify

```bash
# Count BaseModel subclasses in codebase
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model declarations in adapters and services
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify `response_model` parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- Malformed requests to endpoints return 422 Unprocessable Entity with detailed Pydantic validation error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints MUST use Pydantic BaseModel for request/response validation before merge approval.
</enforcement>