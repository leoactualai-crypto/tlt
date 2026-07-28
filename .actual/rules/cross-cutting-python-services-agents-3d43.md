# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Python Services Agents

These rules are ALWAYS ACTIVE for all Python services, agents, adapters, and MCP services within the monorepo that implement logging and domain validation patterns.

### Rules

- **R-LOG-001** MUST: All Python services, agents, and adapters MUST use loguru as the logging framework, imported as `from loguru import logger`.
- **R-LOG-002** MUST: Import loguru logger at module level—avoid lazy imports or conditional logger initialization.
- **R-LOG-003** MUST: Emit `logger.error()` with exception context in all except blocks; include task_id or correlation ID when available.
- **R-LOG-004** MUST: Configure loguru sinks in service `main()` or `__init__.py` to route logs to stdout in JSON format for container log aggregation.
- **R-VAL-001** MUST: All FastAPI endpoint handlers MUST define Pydantic request/response models with Field validators for constrained parameters.
- **R-VAL-002** MUST: Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields—use `ge`/`le` for bounds and `description` for API docs.
- **R-VAL-003** MUST: Use Pydantic `model_validate()` for parsing untrusted input and set `Config.extra='forbid'` to reject unknown fields at API boundaries.
- **R-EXC-001** Exception: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) may use Python standard `logging.getLogger(__name__)` for backward compatibility.

### Verify

```bash
# Count loguru imports in services, agents, and MCP services
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions (excluding tests)
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
- No new standard logging usage detected outside the Discord adapter exception list (EXC-001).

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Pre-commit hooks, CI pipeline checks, and code review checklists enforce loguru usage and Pydantic Field constraints at API boundaries. Violations block pull requests and CI builds unless approved exceptions are documented with EXC-ID references.
</enforcement>