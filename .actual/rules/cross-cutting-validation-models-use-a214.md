# Standardize Pydantic BaseModel for Input Validation in Logged Service Endpoints: Validation Models Use

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that implement logging via `logging.getLogger(__name__)` or `loguru.logger` and accept external input.

### Rules

- **R-VAL-001** MUST: All FastAPI router endpoints that accept external input and implement structured logging SHALL define corresponding Pydantic BaseModel subclasses for request validation before processing.
- **R-VAL-002** MUST: Validation models SHALL include Field constraints (ge, le, min_length, max_length) and description metadata for all fields to enable automatic OpenAPI schema generation and provide context in validation error logs.
- **R-VAL-003** MUST: Validation failures SHALL be logged at error level with sufficient context to correlate with input payloads using `logger.error(f'Validation failed: {e}')` or equivalent.
- **R-VAL-004** MUST: All FastAPI @router decorators for endpoints with logging SHALL specify `response_model=...` parameter to enable automatic response validation and consistent logging of output data.
- **R-VAL-005** SHOULD: Validation models SHOULD be defined in the same module as the router to maintain locality between validation and endpoint logic.
- **R-VAL-006** SHOULD: For nested models (e.g., Optional[List[str]]), define separate BaseModel classes rather than inline type annotations to improve validation error messages.
- **R-VAL-007** MAY: Validation models MAY use Enum types for categorical fields (PhotoQuality, PhotoRelevance) to constrain valid values at the type level.

### Verify

```bash
# Count BaseModel definitions in adapter and service modules
grep -r 'class.*BaseModel' monorepo/tlt/adapters/ monorepo/tlt/services/ | grep -v '__pycache__' | wc -l

# Count @router decorators with response_model parameter
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
- Response models are specified in @router decorators using response_model parameter

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints with logging MUST include Pydantic BaseModel validation before merging. CI pipeline MUST fail if @router decorators lack response_model parameters or if validation models are missing Field constraints for numeric fields.
</enforcement>