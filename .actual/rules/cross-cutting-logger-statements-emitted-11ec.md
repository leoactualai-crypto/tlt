# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Logger Statements Emitted

These rules are ALWAYS ACTIVE for all Python modules in services/, agents/, adapters/, and mcp_services/ directories that implement logging and domain validation patterns, including FastAPI router definitions, agent reasoning nodes, and external integration clients.

### Rules

- **R-LOG-001** MUST: Logger statements MUST be emitted at error boundaries including exception handlers, task failures, health check failures, and external service timeouts.
- **R-LOG-002** MUST: Import loguru logger at module level using `from loguru import logger` — avoid lazy imports or conditional logger initialization.
- **R-LOG-003** MUST: Emit `logger.error()` with exception context in all except blocks, including task_id or correlation ID when available.
- **R-LOG-004** MUST: Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields using ge/le for bounds and description for API docs.
- **R-LOG-005** MUST: Configure loguru sinks in service main() or __init__.py to route logs to stdout in JSON format for container log aggregation.
- **R-LOG-006** MUST: Use Pydantic model_validate() for parsing untrusted input and set Config.extra='forbid' to reject unknown fields at API boundaries.
- **R-LOG-007** SHOULD: Health check endpoints SHOULD emit logger statements for status transitions and include timestamp in ISO 8601 format.
- **R-LOG-008** MAY: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) MAY use Python standard logging.getLogger(__name__) for backward compatibility (EXC-001).

### Verify

```bash
# Count loguru imports in scope directories
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l

# Check for logging anti-patterns using ruff
ruff check --select=G --select=LOG monorepo/tlt/

# Run validation tests with coverage
pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement.
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters.
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format.
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%.
- No new standard logging usage detected outside Discord adapter exception list (EXC-001).

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST run ruff linter to detect standard logging imports in new code outside exception list. CI pipeline MUST run pytest with coverage requirements for Pydantic model validation tests. Code review MUST verify loguru usage and Pydantic Field constraints at API boundaries. Static analysis tools (mypy) MUST enforce Pydantic model type annotations and detect missing Field validators. CI build MUST fail if new standard logging usage detected outside Discord adapter exception list. Pull requests MUST be blocked if Pydantic models lack Field validators for numeric constraints or optional fields.
</enforcement>