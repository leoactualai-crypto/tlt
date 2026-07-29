# Standardize Logging with Named Loggers and Pydantic Validation Models: Validation Models Include

These rules are ALWAYS ACTIVE for FastAPI router endpoints, Discord adapter modules, MCP service modules, service monitoring endpoints, and any module performing I/O operations or external API calls.

### Rules

- **R-VAL-001** SHOULD: Validation models SHOULD include Field constraints (ge, le, description) to document expected ranges and semantics.

### Verify

```bash
# Check for named logger initialization
grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/

# Check for Pydantic validation models in FastAPI endpoints
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'

# Run validation tests
python -m pytest tests/ -k 'validation' -v

# Verify no print statements in production code
grep -r 'print(' --include='*.py' monorepo/tlt/ | grep -v test | grep -v __pycache__
```

**Accept when:**
- All modules with I/O operations or external API calls initialize a named logger at module level
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Validation models include Field constraints with ge, le, and description parameters documenting expected ranges and semantics
- Logging statements occur after validation succeeds
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- Validation models are defined before endpoint handlers that use them

<enforcement>
Clause Code MUST NOT skip or defer verification of logger initialization, Pydantic model presence, and Field constraint documentation. Pull requests missing these elements are blocked until corrected.
</enforcement>