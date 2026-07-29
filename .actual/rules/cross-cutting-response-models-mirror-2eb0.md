# Standardize Pydantic BaseModel for Input Validation in Logged Service Endpoints: Response Models Mirror

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that implement logging via `logging.getLogger(__name__)` or `loguru.logger` and accept external input.

### Rules

- **R-PYDANTIC-001** SHOULD: Response models SHOULD mirror request models with additional fields (id, status, created_at, updated_at) to maintain validation consistency across request/response cycles.

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
- Response models are specified in @router decorators using response_model parameter

<enforcement>
Clause Code MUST NOT skip or defer verification. All new FastAPI endpoints with logging require Pydantic BaseModel definitions and response_model parameters. CI pipeline fails if these requirements are not met.
</enforcement>