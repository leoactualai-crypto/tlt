# Enforce Pydantic BaseModel Validation for All Input Data Models: Models Include Field

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent and timer context models, and configuration models that process external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-001** MUST: All models accepting external input, API requests, event payloads, or user-submitted data MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-002** SHOULD: Models SHOULD include Field descriptions for all fields to document validation intent and improve API documentation.
- **R-PYDANTIC-003** MUST: Numeric fields with semantic constraints (scores, ratings, percentages, bounds) MUST use Field validators with ge/le bounds (e.g., Field(ge=0.0, le=1.0) for normalized scores).
- **R-PYDANTIC-004** MUST: Collection fields (Dict, List) MUST consistently use Field(default_factory) to prevent mutable default bugs across instances.
- **R-PYDANTIC-005** MUST: Timestamp fields MUST use timezone-aware datetime with default_factory for UTC timestamp generation (e.g., Field(default_factory=lambda: datetime.now(timezone.utc))).
- **R-PYDANTIC-006** MUST: All nested context models (DiscordContext, TimerContext, EventContext) MUST be defined as separate BaseModel classes for reusability and clear validation boundaries.

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
- All model files in mcp_services/*/models.py, agents/*/state/*.py, and adapters/*/models.py define input models inheriting from Pydantic BaseModel
- Numeric fields with semantic constraints (scores, ratings, percentages) use Field validators with ge/le bounds
- All timestamp fields use timezone-aware datetime with default_factory for UTC timestamp generation
- Collection fields (Dict, List) consistently use Field(default_factory) to prevent mutable default bugs
- Nested context models are defined as separate BaseModel classes with clear validation boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All new models accepting external input MUST inherit from BaseModel. Code review MUST block merge if validation constraints are missing for numeric fields with semantic bounds. CI pipeline MUST fail if new models do not meet these requirements.
</enforcement>