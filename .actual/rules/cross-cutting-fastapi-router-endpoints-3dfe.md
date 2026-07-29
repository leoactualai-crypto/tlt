# Standardize Pydantic BaseModel for Input Validation in Logged Service Endpoints: Fastapi Router Endpoints

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that implement logging via `logging.getLogger(__name__)` or `loguru.logger` and accept external input.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoints that implement structured logging via `logging.getLogger(__name__)` or `loguru.logger` MUST define Pydantic BaseModel subclasses for request and response payloads.
- **R-PYDANTIC-002** MUST: Define BaseModel subclasses in the same module as the router to maintain locality between validation and endpoint logic.
- **R-PYDANTIC-003** MUST: Use `Field(description=...)` for all fields to generate self-documenting OpenAPI schemas and provide context in validation error logs.
- **R-PYDANTIC-004** MUST: Log validation failures at error level with logger.error(f'Validation failed: {e}') to distinguish from business logic errors.
- **R-PYDANTIC-005** MUST: For nested models (e.g., `Optional[List[str]]`), define separate BaseModel classes rather than inline type annotations to improve validation error messages.
- **R-PYDANTIC-006** MUST: Include `response_model=...` in `@router` decorators to enable automatic response validation and consistent logging of output data.

### Verify

```bash
# Count BaseModel definitions in adapter and service modules
grep -r 'class.*BaseModel' monorepo/tlt/adapters/ monorepo/tlt/services/ | grep -v '__pycache__' | wc -l

# Count router decorators with response_model parameter
grep -r '@router\.(get|post|put|delete)' monorepo/tlt/ | grep 'response_model=' | wc -l

# Count Field constraints and descriptions
grep -r 'Field(' monorepo/tlt/ | grep -E '(ge=|le=|description=)' | wc -l

# Count logging imports
grep -r 'logging\.getLogger\|from loguru import logger' monorepo/tlt/ | wc -l
```

**Accept when:**
- All FastAPI router endpoints that implement logging have corresponding Pydantic BaseModel definitions for request/response payloads
- At least 80% of BaseModel fields include Field constraints or description metadata
- Validation failures are logged at error level with sufficient context to correlate with input payloads
- Response models are specified in `@router` decorators using `response_model` parameter

<enforcement>
Clause Code MUST NOT skip or defer verification. All new FastAPI endpoints with logging require Pydantic BaseModel definitions; CI pipeline fails if missing. Code review blocks merge if validation models lack Field constraints for numeric fields. Runtime validation errors are logged and monitored; repeated failures trigger alerts.
</enforcement>