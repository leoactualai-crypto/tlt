# Standardize Logging with Named Loggers and Pydantic Validation Models: Error Conditions Logged

These rules are ALWAYS ACTIVE for FastAPI router endpoints, Discord adapter modules, MCP service modules, service monitoring endpoints, and any module performing I/O operations or external API calls.

### Rules

- **R-LOG-001** SHOULD: Error conditions SHOULD be logged with logger.error() or equivalent before raising HTTPException or returning error responses.

### Verify

```bash
# Check for named logger initialization
grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/

# Check for Pydantic validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'

# Run validation tests
python -m pytest tests/ -k 'validation' -v
```

**Accept when:**
- All modules with I/O operations or external API calls initialize a named logger at module level
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Error logging statements occur before HTTPException is raised or error responses are returned
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- Logger initialization uses `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports

<enforcement>
Clause R-LOG-001 verification is mandatory. Code review must confirm error conditions are logged before exceptions are raised. Static analysis must detect missing logger initialization in I/O-performing modules. CI pipeline must block pull requests missing logger initialization or validation models.
</enforcement>