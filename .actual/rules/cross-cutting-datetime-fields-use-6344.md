# Enforce Pydantic BaseModel Validation for All Input Data Models: Datetime Fields Use

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent and timer context models, and configuration models that process external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-DT-001** MUST: DateTime fields MUST use timezone-aware datetime objects with `default_factory=lambda: datetime.now(timezone.utc)` for timestamp generation.
- **R-PYDANTIC-DT-002** MUST: All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` that accept external input MUST define input models inheriting from Pydantic `BaseModel`.
- **R-PYDANTIC-DT-003** MUST: Numeric fields with semantic constraints (scores, ratings, percentages) MUST use Field validators with `ge`/`le` bounds (e.g., `Field(ge=0.0, le=1.0)` for normalized scores).
- **R-PYDANTIC-DT-004** MUST: Collection fields (Dict, List) MUST consistently use `Field(default_factory)` to prevent mutable default bugs across instances.

### Verify

```bash
# Count BaseModel definitions in model files
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l

# Verify Field constraints on numeric fields
grep -r 'Field(ge=' monorepo/tlt --include='*.py' | grep -E '(quality_score|relevance_score|vibe_score|confidence_score)' | wc -l

# Verify timezone-aware datetime defaults
grep -r 'default_factory=lambda: datetime.now(timezone.utc)' monorepo/tlt --include='*.py' | wc -l

# Verify BaseModel inheritance in model files
python -c 'from pydantic import BaseModel, Field; import ast; import sys; [print(f"PASS: {f}") for f in sys.argv[1:] if any("BaseModel" in n.name for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.Name))]' monorepo/tlt/mcp_services/*/models.py
```

**Accept when:**
- All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` define input models inheriting from Pydantic BaseModel
- Numeric fields with semantic constraints (scores, ratings, percentages) use Field validators with `ge`/`le` bounds
- All timestamp fields use timezone-aware datetime with `default_factory` for UTC timestamp generation
- Collection fields (Dict, List) consistently use `Field(default_factory)` to prevent mutable default bugs

<enforcement>
Claude Code MUST NOT skip or defer verification. All datetime fields in input models MUST use timezone-aware UTC timestamps with the specified default_factory pattern. Code review MUST block merge if validation constraints are missing for numeric fields with semantic bounds. CI pipeline MUST fail if new models accepting external input do not inherit from BaseModel.
</enforcement>