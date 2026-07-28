# Enforce Pydantic BaseModel Validation for All Input Data Models: Numeric Fields Semantic

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent models, timer context models, and configuration models that accept external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-001** MUST: All numeric fields with semantic constraints (scores, ratings, percentages, confidence values) MUST use Pydantic Field validators with bounds enforcement using `ge`, `le`, `gt`, or `lt` parameters (e.g., `Field(ge=0.0, le=1.0)` for normalized scores, `Field(ge=1, le=5)` for rating scales).
- **R-PYDANTIC-002** MUST: All mutable collection fields (Dict, List) MUST use `Field(default_factory=dict)` or `Field(default_factory=list)` to prevent shared mutable default bugs across model instances.
- **R-PYDANTIC-003** MUST: All timestamp fields MUST use timezone-aware datetime with `Field(default_factory=lambda: datetime.now(timezone.utc))` to ensure UTC timestamp generation, as demonstrated in RSVPCreate, EventCreate, and AgentTask models.
- **R-PYDANTIC-004** MUST: Nested context models (DiscordContext, TimerContext, EventContext) MUST be defined as separate BaseModel classes for reusability and clear validation boundaries.

### Verify

```bash
# Count BaseModel definitions in model files
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l

# Verify numeric field constraints are present
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
- Nested context models are defined as separate BaseModel classes with clear validation boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All numeric field constraints MUST be validated before accepting model definitions. Validation errors at API boundaries MUST be properly sanitized before returning to clients.
</enforcement>