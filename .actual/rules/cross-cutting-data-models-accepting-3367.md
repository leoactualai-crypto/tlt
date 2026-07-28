# Enforce Pydantic BaseModel Validation for All Input Data Models: Data Models Accepting

These rules are ALWAYS ACTIVE for all services, agents, and adapters that process external input, API requests, event payloads, or user-submitted data. All data models accepting untrusted input MUST use Pydantic BaseModel validation.

### Rules

- **R-PYDANTIC-001** MUST: All data models accepting external input (API requests, event payloads, user submissions, configuration) MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-002** MUST: Numeric fields with semantic constraints (scores, ratings, percentages) MUST use Field validators with ge/le bounds (e.g., Field(ge=0.0, le=1.0) for normalized scores, Field(ge=1, le=5) for rating scales).
- **R-PYDANTIC-003** MUST: Collection fields (Dict, List) MUST consistently use Field(default_factory) to prevent mutable default bugs across instances.
- **R-PYDANTIC-004** MUST: Timestamp fields MUST use timezone-aware datetime with default_factory for UTC timestamp generation (e.g., Field(default_factory=lambda: datetime.now(timezone.utc))).
- **R-PYDANTIC-005** SHOULD: Define nested context models (DiscordContext, TimerContext, EventContext) as separate BaseModel classes for reusability and clear validation boundaries.

### Verify

```bash
# Count BaseModel definitions in model files
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l

# Verify Field constraints on score fields
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
- No raw dict or list usage at API boundaries for external input models

<enforcement>
Clause Code MUST NOT skip or defer verification. All new data models accepting external input MUST inherit from Pydantic BaseModel. CI pipeline MUST fail if validation constraints are missing for numeric fields with semantic bounds. Code review MUST block merge if validation requirements are not met. Security review is required for any code bypassing validation at trust boundaries.
</enforcement>