# Standardize Logging with Named Loggers and Pydantic Validation Models: Logging Statements Occur

These rules are ALWAYS ACTIVE for FastAPI router endpoints, Discord adapter modules, MCP service modules, service monitoring endpoints, and any module that performs I/O operations or external API calls.

### Rules

- **R-LOG-001** MUST: Logging statements MUST occur after input validation has succeeded, not before validation.
- **R-LOG-002** MUST: All modules with I/O operations or external API calls MUST initialize a named logger using `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports.
- **R-LOG-003** MUST: All FastAPI router endpoints MUST define Pydantic BaseModel validation classes for request and response contracts.
- **R-LOG-004** MUST: No print() statements MUST exist in production code paths (excluding explicitly documented exceptions EXC-001 and EXC-002).
- **R-LOG-005** SHOULD: Use logger.info() for successful operations, logger.error() for failures, and include contextual data (user_id, message_id, etc.) in log messages.

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
- All modules with I/O operations or external API calls initialize a named logger
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Logging statements occur after validation succeeds, verified by code review or static analysis
- No print() statements exist in production code paths (excluding explicitly documented exceptions)
- Logger initialization and validation model presence are confirmed in code review
- Static analysis with grep or AST-based linting detects no missing loggers or validation models
- CI pipeline checks confirm no print() statements in non-test code

<enforcement>
Clause Code MUST NOT skip or defer verification of logging statement placement relative to validation, named logger initialization, Pydantic model definitions, and absence of print() statements in production code.
</enforcement>