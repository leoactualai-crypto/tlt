# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Logger Statements Include

These rules are ALWAYS ACTIVE for all Python modules in `monorepo/tlt/services/`, `monorepo/tlt/agents/`, `monorepo/tlt/adapters/`, and `monorepo/tlt/mcp_services/`, including FastAPI router definitions, agent reasoning nodes, and external integration clients.

### Rules

- **R-LOG-001** SHOULD: Logger statements SHOULD include structured context such as task_id, event_id, user_id, or correlation identifiers to enable trace reconstruction.
- **R-LOG-002** MUST: Import loguru logger at module level using `from loguru import logger` — avoid lazy imports or conditional logger initialization.
- **R-LOG-003** MUST: Emit `logger.error()` with exception context in all except blocks, including task_id or correlation ID when available.
- **R-LOG-004** SHOULD: Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields — use ge/le for bounds, description for API docs.
- **R-LOG-005** SHOULD: Configure loguru sinks in service main() or __init__.py to route logs to stdout in JSON format for container log aggregation.
- **R-LOG-006** SHOULD: Use Pydantic model_validate() for parsing untrusted input and Config.extra='forbid' to reject unknown fields at API boundaries.
- **R-LOG-007** MUST: Do not use Python standard logging (logging.getLogger(__name__)) in new code outside the documented exception list (EXC-001: Legacy Discord adapter modules).

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

# Detect standard logging usage outside exceptions
grep -r 'logging.getLogger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | grep -v 'event.py\|reminder.py\|experience_manager.py\|rsvp.py'
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement with structured context.
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters.
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format.
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%.
- No new standard logging usage detected outside the documented exception list (EXC-001).

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Pre-commit hooks, CI builds, and code review checklists MUST validate compliance before merge.
</enforcement>