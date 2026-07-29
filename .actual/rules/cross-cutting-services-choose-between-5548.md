# Standardize Logging with Named Loggers and Pydantic Validation Models: Services Choose Between

These rules are ALWAYS ACTIVE for FastAPI router endpoints, Discord adapter modules, MCP service modules, service monitoring endpoints, and any module performing I/O operations or external API calls.

### Rules

- **R-LOG-001** MUST: Services MUST choose between standard library logging and loguru based on feature requirements, but MUST NOT mix both within a single module.
- **R-LOG-002** MUST: Initialize loggers at module level using `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports.
- **R-LOG-003** MUST: Define Pydantic BaseModel classes for all FastAPI router request and response types, using Field() for constraints and documentation.
- **R-LOG-004** MUST: Structure endpoint handlers to validate inputs first (via FastAPI dependency injection), then log operations, then execute business logic.
- **R-LOG-005** MUST: Use logger.info() for successful operations, logger.error() for failures, and include contextual data (user_id, message_id, etc.) in log messages.
- **R-LOG-006** MUST: Configure handlers and formatters in the service entry point (main.py) to ensure consistent output format for modules using standard logging.
- **R-LOG-007** MAY: Legacy modules undergoing gradual migration may temporarily use print() statements (EXC-001).
- **R-LOG-008** MAY: Performance-critical hot paths may defer validation to reduce latency (EXC-002).

### Verify

```bash
# Verify named logger initialization
grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/

# Verify Pydantic validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'

# Verify validation tests
python -m pytest tests/ -k 'validation' -v

# Detect mixed logging libraries in single modules
for file in $(find monorepo/tlt -name '*.py' -type f); do
  if grep -q 'logging.getLogger' "$file" && grep -q 'from loguru import logger' "$file"; then
    echo "VIOLATION: Mixed logging in $file"
  fi
done

# Detect print statements in production code
grep -r 'print(' --include='*.py' monorepo/tlt/ | grep -v test | grep -v '#'
```

**Accept when:**
- All modules with I/O operations or external API calls initialize a named logger
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Logging statements occur after validation succeeds, verified by code review or static analysis
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- No single module mixes logging.getLogger and loguru imports
- Logger initialization occurs immediately after imports at module level
- Contextual data (user_id, message_id, etc.) is included in log messages for operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for modules in scope. Violations must be tracked and remediated according to the exception process defined in the ADR.
</enforcement>