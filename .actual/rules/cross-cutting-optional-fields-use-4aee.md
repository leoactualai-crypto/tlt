# Enforce Pydantic BaseModel Validation for All Input Data Models: Optional Fields Use

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent models, timer context models, and configuration models that accept external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-001** MUST: Optional fields MUST use `Optional[T]` type hints and provide explicit default values or `default_factory` functions.
- **R-PYDANTIC-002** MUST: All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` define input models inheriting from Pydantic `BaseModel`.
- **R-PYDANTIC-003** MUST: Numeric fields with semantic constraints (scores, ratings, percentages) use `Field` validators with `ge`/`le` bounds.
- **R-PYDANTIC-004** MUST: All timestamp fields use timezone-aware `datetime` with `default_factory` for UTC timestamp generation.
- **R-PYDANTIC-005** MUST: Collection fields (`Dict`, `List`) consistently use `Field(default_factory)` to prevent mutable default bugs.
- **R-PYDANTIC-006** SHOULD: Use `Field(ge=0.0, le=1.0)` for normalized scores and percentages, `Field(ge=1, le=5)` for rating scales.
- **R-PYDANTIC-007** SHOULD: Define nested context models (DiscordContext, TimerContext, EventContext) as separate `BaseModel` classes for reusability and clear validation boundaries.

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
- All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` define input models inheriting from Pydantic `BaseModel`
- Numeric fields with semantic constraints (scores, ratings, percentages) use `Field` validators with `ge`/`le` bounds
- All timestamp fields use timezone-aware `datetime` with `default_factory` for UTC timestamp generation
- Collection fields (`Dict`, `List`) consistently use `Field(default_factory)` to prevent mutable default bugs
- Optional fields consistently use `Optional[T]` type hints with explicit defaults or `default_factory` functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All new models accepting external input MUST inherit from Pydantic BaseModel with proper Field constraints and default factories. Code review MUST block merge if validation constraints are missing for numeric fields with semantic bounds. CI pipeline MUST fail if new models do not meet these requirements.
</enforcement>