# Standardize Logging with Named Loggers and Pydantic Validation Models: Modules That Perform

These rules are ALWAYS ACTIVE for all modules that perform I/O operations, external API calls, handle bot events and commands, process external data, or implement FastAPI router endpoints accepting external requests.

### Rules

- **R-LOG-001** MUST: All modules that perform logging MUST initialize a named logger using either `logging.getLogger(__name__)` or `from loguru import logger` at module level immediately after imports.
- **R-LOG-002** MUST: All FastAPI router endpoints accepting external requests MUST define Pydantic BaseModel validation classes for request and response types.
- **R-LOG-003** MUST: Endpoint handlers MUST structure operations to validate inputs first (via FastAPI dependency injection), then log operations, then execute business logic.
- **R-LOG-004** MUST: No `print()` statements MUST exist in production code paths (excluding explicitly documented exceptions EXC-001 and EXC-002).
- **R-LOG-005** SHOULD: Use `logger.info()` for successful operations and `logger.error()` for failures, including contextual data (user_id, message_id, etc.) in log messages.
- **R-LOG-006** SHOULD: For modules using standard logging, configure handlers and formatters in the service entry point (main.py) to ensure consistent output format.

### Verify

```bash
# Verify named logger initialization
grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/

# Verify Pydantic validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'

# Verify validation tests
python -m pytest tests/ -k 'validation' -v

# Verify no print statements in production code
grep -r 'print(' --include='*.py' monorepo/tlt/ | grep -v test | grep -v __pycache__
```

**Accept when:**
- All modules with I/O operations or external API calls initialize a named logger at module level
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Logging statements occur after validation succeeds, verified by code review or static analysis
- No `print()` statements exist in production code paths (excluding explicitly documented exceptions)
- Logger initialization uses either `logging.getLogger(__name__)` or `from loguru import logger`
- Contextual data (user_id, message_id, etc.) is included in log messages for operations

<enforcement>
Clause Code MUST NOT skip or defer verification of logger initialization, validation model presence, and absence of print() statements in production code. Pull requests missing logger initialization or validation models MUST be blocked until corrected. Repeated violations trigger architecture review and additional training.
</enforcement>