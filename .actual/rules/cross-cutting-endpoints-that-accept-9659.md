# Standardize Logging with Named Loggers and Pydantic Validation Models: Endpoints That Accept

These rules are ALWAYS ACTIVE for all FastAPI router endpoints, Discord adapter modules, MCP service modules, and any code performing I/O operations or external API calls that accept external input.

### Rules

- **R-LOG-001** MUST: API endpoints that accept external input MUST define Pydantic BaseModel validation classes for request and response contracts.
- **R-LOG-002** MUST: Initialize loggers at module level using `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports in all modules with I/O operations or external API calls.
- **R-LOG-003** MUST: Structure endpoint handlers to validate inputs first (via FastAPI dependency injection), then log operations, then execute business logic.
- **R-LOG-004** MUST: Use logger.info() for successful operations and logger.error() for failures, including contextual data (user_id, message_id, etc.) in log messages.
- **R-LOG-005** MUST: No print() statements shall exist in production code paths (excluding explicitly documented exceptions EXC-001 and EXC-002).
- **R-LOG-006** SHOULD: Define Pydantic BaseModel classes using Field() for constraints and documentation on all FastAPI router request and response types.
- **R-LOG-007** SHOULD: Configure handlers and formatters in the service entry point (main.py) for modules using standard logging to ensure consistent output format.

### Verify

```bash
# Verify named logger initialization
grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/

# Verify Pydantic validation models exist
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'

# Verify validation tests pass
python -m pytest tests/ -k 'validation' -v

# Verify no print statements in production code
grep -r 'print(' --include='*.py' monorepo/tlt/ | grep -v test | grep -v __pycache__
```

**Accept when:**
- All modules with I/O operations or external API calls initialize a named logger at module level
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request and response contracts
- Logging statements occur after validation succeeds, verified by code review or static analysis
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- Validation tests pass with 100% coverage of request/response models
- Logger initialization appears immediately after imports in all applicable modules

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-LOG-### rules marked MUST are non-negotiable. Pull requests missing logger initialization or validation models MUST be blocked until corrected. Print statements in production code MUST trigger CI failure.
</enforcement>