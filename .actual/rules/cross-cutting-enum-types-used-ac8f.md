# Enforce Pydantic BaseModel Validation for All Input Data Models: Enum Types Used

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent and timer context models, and configuration models that process external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-001** MUST: All data models accepting untrusted input (external APIs, events, user submissions) inherit from Pydantic `BaseModel`.
- **R-PYDANTIC-002** SHOULD: Enum types SHOULD be used for fields with fixed value sets (status, priority, role) to enforce allowed values at the type level.
- **R-PYDANTIC-003** MUST: Numeric fields with semantic constraints (scores, ratings, percentages) use `Field(ge=..., le=...)` validators to enforce bounds.
- **R-PYDANTIC-004** MUST: All timestamp fields use timezone-aware `datetime` with `Field(default_factory=lambda: datetime.now(timezone.utc))` for UTC generation.
- **R-PYDANTIC-005** MUST: Collection fields (Dict, List) consistently use `Field(default_factory=dict)` or `Field(default_factory=list)` to prevent mutable default bugs.
- **R-PYDANTIC-006** MUST: Validation occurs at trust boundaries (API endpoints, event handlers, external data sources) before data enters business logic.

### Verify

```bash
# Count BaseModel usage across model files
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l

# Verify Field constraints on score fields
grep -r 'Field(ge=' monorepo/tlt --include='*.py' | grep -E '(quality_score|relevance_score|vibe_score|confidence_score)' | wc -l

# Verify timezone-aware datetime defaults
grep -r 'default_factory=lambda: datetime.now(timezone.utc)' monorepo/tlt --include='*.py' | wc -l

# Verify BaseModel inheritance in model files
python -c 'from pydantic import BaseModel, Field; import ast; import sys; [print(f"PASS: {f}") for f in sys.argv[1:] if any("BaseModel" in n.name for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.Name))]' monorepo/tlt/mcp_services/*/models.py
```

**Accept when:**
- All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` define input models inheriting from Pydantic `BaseModel`
- Numeric fields with semantic constraints (scores, ratings, percentages) use `Field` validators with `ge`/`le` bounds
- All timestamp fields use timezone-aware `datetime` with `default_factory` for UTC timestamp generation
- Collection fields (Dict, List) consistently use `Field(default_factory)` to prevent mutable default bugs
- Enum types are used for fields with fixed value sets (status, priority, role)
- Validation errors at API boundaries are properly sanitized before returning to clients

<enforcement>
Claude Code MUST NOT skip or defer verification. All new models accepting external input MUST inherit from Pydantic BaseModel. Code review MUST block merge if validation constraints are missing for numeric fields with semantic bounds. CI pipeline MUST fail if models do not meet these requirements.
</enforcement>