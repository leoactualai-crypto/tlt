# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Pydantic Models Include

These rules are ALWAYS ACTIVE for all Python modules in `monorepo/tlt/services/`, `monorepo/tlt/agents/`, `monorepo/tlt/adapters/`, and `monorepo/tlt/mcp_services/`, including FastAPI router definitions, agent reasoning nodes, and external integration clients.

### Rules

- **R-LOGURU-PYDANTIC-001** SHOULD: Pydantic models SHOULD include Field constraints (ge, le, description) for numeric scores, confidence values, and bounded parameters.

### Verify

```bash
# Count loguru imports across services, agents, and mcp_services
grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l

# Count Pydantic BaseModel definitions (excluding tests)
grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l

# Check for logging anti-patterns using ruff
ruff check --select=G --select=LOG monorepo/tlt/

# Run validation tests with coverage
pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing
```

**Accept when:**
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%
- All numeric score fields (confidence, probability, rating) include Field constraints with ge/le bounds and description text

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks and CI pipeline enforce loguru usage and Pydantic Field constraints. Pull requests are blocked if Pydantic models lack Field validators for numeric constraints. Violations outside the Discord adapter exception list (EXC-001) cause CI build failure.
</enforcement>