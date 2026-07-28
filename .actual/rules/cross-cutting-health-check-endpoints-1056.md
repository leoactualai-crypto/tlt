# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Health Check Endpoints

These rules are ALWAYS ACTIVE for all Python modules in `monorepo/tlt/services/`, `monorepo/tlt/agents/`, `monorepo/tlt/adapters/`, and `monorepo/tlt/mcp_services/` that implement logging and domain validation patterns, including FastAPI router definitions, endpoint handlers, agent reasoning nodes, and external integration clients.

### Rules

- **R-LOGURU-001** MUST: Import loguru logger at module level using `from loguru import logger` — avoid lazy imports or conditional logger initialization.
- **R-LOGURU-002** MUST: Health check endpoints MUST log status transitions (healthy, degraded, unhealthy) with timestamp in ISO 8601 format and diagnostic context.
- **R-LOGURU-003** MUST: Emit `logger.error()` with exception context in all except blocks, including task_id or correlation ID when available.
- **R-LOGURU-004** SHOULD: Configure loguru sinks in service `main()` or `__init__.py` to route logs to stdout in JSON format for container log aggregation.
- **R-PYDANTIC-001** MUST: Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields using `ge`/`le` for bounds and `description` for API docs.
- **R-PYDANTIC-002** MUST: All FastAPI endpoint handlers MUST define Pydantic request/response models with Field validators for constrained parameters.
- **R-PYDANTIC-003** MUST: Use Pydantic `model_validate()` for parsing untrusted input and set `Config.extra='forbid'` to reject unknown fields at API boundaries.
- **R-VALIDATION-001** SHOULD: Implement FastAPI exception handlers to sanitize Pydantic ValidationError responses and audit error messages for information disclosure.
- **R-EXCEPTION-001** MUST: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) MAY use Python standard `logging.getLogger(__name__)` for backward compatibility under exception EXC-001.

### Verify

```bash
# Count loguru imports in scope
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l

# Check for logging anti-patterns
ruff check --select=G --select=LOG monorepo/tlt/

# Run validation tests with coverage
pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing

# Verify no standard logging outside exception list
grep -r 'import logging' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | grep -v 'event.py\|reminder.py\|experience_manager.py\|rsvp.py' | wc -l
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%
- No new standard logging usage detected outside Discord adapter exception list (EXC-001)
- All Pydantic models include Field validators for numeric constraints (ge/le) and optional fields

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST run ruff linter to detect standard logging imports outside exception list. CI pipeline MUST run pytest with coverage requirements for Pydantic model validation tests. Code review MUST verify loguru usage and Pydantic Field constraints at API boundaries. Pull requests MUST be blocked if Pydantic models lack Field validators or new standard logging usage is detected outside exceptions. Violations result in CI build failure and require exception board approval.
</enforcement>