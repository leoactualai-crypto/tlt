# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Services Use Python

These rules are ALWAYS ACTIVE for all Python modules in services/, agents/, adapters/, and mcp_services/ directories that implement logging and domain validation patterns.

### Rules

- **R-LOG-001** MUST: Use loguru logger for all new Python modules in services/, agents/, adapters/, and mcp_services/. Import at module level: `from loguru import logger`.
- **R-LOG-002** MUST NOT: Mix loguru and Python standard logging within the same module.
- **R-LOG-003** MAY: Use Python's standard logging module for legacy integrations, but MUST NOT mix loguru and standard logging within the same module.
- **R-VAL-001** MUST: Define Pydantic BaseModel for all FastAPI endpoint request/response handlers with explicit Field validators for constrained parameters.
- **R-VAL-002** MUST: Use Field constraints (ge/le for numeric bounds, pattern for strings, description for API docs) on all Pydantic model fields representing domain boundaries.
- **R-VAL-003** MUST: Use Pydantic model_validate() for parsing untrusted input and set Config.extra='forbid' to reject unknown fields at API boundaries.
- **R-LOG-004** MUST: Emit logger.error() with exception context in all except blocks, including task_id or correlation ID when available.
- **R-LOG-005** MUST: Configure loguru sinks in service main() or __init__.py to route logs to stdout in JSON format for container log aggregation.
- **R-LOG-006** MUST: Include timestamp in ISO 8601 format in health check endpoint log statements for status transitions.
- **R-EXC-001** EXCEPTION: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) MAY use Python standard logging.getLogger(__name__) for backward compatibility.

### Verify

```bash
# Count loguru imports in service modules
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l

# Check for logging anti-patterns
ruff check --select=G --select=LOG monorepo/tlt/

# Run validation tests with coverage
pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing

# Detect standard logging usage outside exception list
grep -r 'import logging' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | grep -v 'event.py\|reminder.py\|experience_manager.py\|rsvp.py'

# Verify Pydantic Field validators on numeric constraints
grep -r 'Field.*ge=\|Field.*le=' monorepo/tlt/ | wc -l
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%
- No new standard logging usage detected outside Discord adapter exception list (EXC-001)
- All Pydantic models at API boundaries use model_validate() and Config.extra='forbid'

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST run ruff linter to detect standard logging imports in new code outside exception list. CI pipeline MUST run pytest with coverage requirements for Pydantic model validation tests. Code review checklist MUST include verification of loguru usage and Pydantic Field constraints at API boundaries. Pull requests MUST be blocked if Pydantic models lack Field validators for numeric constraints or optional fields.
</enforcement>