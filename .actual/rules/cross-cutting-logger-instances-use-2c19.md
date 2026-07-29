# Standardize Logging with Named Loggers and Pydantic Validation Models: Logger Instances Use

These rules are ALWAYS ACTIVE for FastAPI router endpoints, Discord adapter modules, MCP service modules, service monitoring endpoints, and any module performing I/O operations or external API calls.

### Rules

- **R-LOG-001** SHOULD: Logger instances SHOULD use module-level `__name__` to enable hierarchical logger configuration and filtering.
- **R-LOG-002** MUST: Initialize loggers at module level using `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports.
- **R-LOG-003** MUST: Define Pydantic BaseModel classes for all FastAPI router request and response types, using Field() for constraints and documentation.
- **R-LOG-004** MUST: Structure endpoint handlers to validate inputs first (via FastAPI dependency injection), then log operations, then execute business logic.
- **R-LOG-005** SHOULD: Use logger.info() for successful operations, logger.error() for failures, and include contextual data (user_id, message_id, etc.) in log messages.
- **R-LOG-006** MUST: Configure handlers and formatters in the service entry point (main.py) to ensure consistent output format for modules using standard logging.
- **R-LOG-007** MUST: Ensure no print() statements exist in production code paths (excluding explicitly documented exceptions EXC-001 and EXC-002).

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
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- Logger initialization uses `__name__` for hierarchical configuration
- Validation models use Field() for constraints and documentation
- Endpoint handlers follow the pattern: validate → log → execute business logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review acceptance. Violations must be corrected before merge, or an approved exception (EXC-001 or EXC-002) must be documented in the module docstring with expiration date or migration plan.
</enforcement>