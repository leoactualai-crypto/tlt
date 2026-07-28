# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Endpoints Agent Nodes

These rules are ALWAYS ACTIVE for all Python modules in services/, agents/, adapters/, and mcp_services/ directories that implement logging and domain validation patterns, including FastAPI endpoint handlers, agent reasoning nodes, and external integration clients.

### Rules

- **R-LOG-001** MUST: All API endpoints, agent nodes, and external integration points MUST define domain models using Pydantic BaseModel with Field validators for input/output contracts.
- **R-LOG-002** MUST: Import loguru logger at module level using `from loguru import logger` — avoid lazy imports or conditional logger initialization.
- **R-LOG-003** MUST: Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields — use ge/le for bounds, description for API docs.
- **R-LOG-004** MUST: Emit logger.error() with exception context in all except blocks, including task_id or correlation ID when available.
- **R-LOG-005** MUST: Configure loguru sinks in service main() or __init__.py to route logs to stdout in JSON format for container log aggregation.
- **R-LOG-006** MUST: Use Pydantic model_validate() for parsing untrusted input and Config.extra='forbid' to reject unknown fields at API boundaries.
- **R-LOG-007** MUST: Health check endpoints MUST emit logger statements for status transitions and include timestamp in ISO 8601 format.
- **R-LOG-008** SHOULD: Implement FastAPI exception handlers to sanitize Pydantic ValidationError responses and audit error messages for information disclosure.
- **R-LOG-009** MAY: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) may use Python standard logging.getLogger(__name__) for backward compatibility (EXC-001).

### Verify

```bash
# Count loguru imports in scope directories
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions
grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l

# Check for logging anti-patterns
ruff check --select=G --select=LOG monorepo/tlt/

# Run validation tests with coverage
pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing

# Detect standard logging usage outside exception list
grep -r 'logging.getLogger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | grep -v 'event.py\|reminder.py\|experience_manager.py\|rsvp.py'
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement.
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters.
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format.
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%.
- No new standard logging usage detected outside Discord adapter exception list (EXC-001).
- All Pydantic models use model_validate() for untrusted input and Config.extra='forbid' at API boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval. CI pipeline must enforce via pre-commit hooks (ruff linter), pytest coverage gates, and static analysis (mypy). Violations block pull requests unless approved exception request is submitted to architecture review board with documented justification and migration timeline.
</enforcement>