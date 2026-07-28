# Enforce Pydantic BaseModel Validation for All Input Data Models: Collection Fields Dict

These rules are ALWAYS ACTIVE for all MCP service models, agent state models, adapter models, API request/response models, CloudEvent models, timer context models, and configuration models that accept external input, API requests, event payloads, or user-submitted data.

### Rules

- **R-PYDANTIC-001** MUST: Collection fields (Dict, List) MUST use `Field(default_factory=dict)` or `Field(default_factory=list)` to prevent mutable default argument bugs.

### Verify

```bash
# Count BaseModel definitions in model files
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l

# Verify numeric fields use Field constraints
grep -r 'Field(ge=' monorepo/tlt --include='*.py' | grep -E '(quality_score|relevance_score|vibe_score|confidence_score)' | wc -l

# Verify timestamp fields use timezone-aware UTC defaults
grep -r 'default_factory=lambda: datetime.now(timezone.utc)' monorepo/tlt --include='*.py' | wc -l

# Verify BaseModel inheritance in model files
python -c 'from pydantic import BaseModel, Field; import ast; import sys; [print(f"PASS: {f}") for f in sys.argv[1:] if any("BaseModel" in n.name for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.Name))]' monorepo/tlt/mcp_services/*/models.py

# Check for collection fields without default_factory
grep -r 'Dict\|List' monorepo/tlt --include='*.py' | grep -v 'default_factory' | grep -E '(models\.py|state\.py)' || echo "No unprotected collection fields found"
```

**Accept when:**
- All model files in `mcp_services/*/models.py`, `agents/*/state/*.py`, and `adapters/*/models.py` define input models inheriting from Pydantic BaseModel
- Numeric fields with semantic constraints (scores, ratings, percentages) use Field validators with `ge`/`le` bounds
- All timestamp fields use timezone-aware datetime with `default_factory` for UTC timestamp generation
- Collection fields (Dict, List) consistently use `Field(default_factory)` to prevent mutable default bugs
- No collection fields in input models lack explicit `default_factory` specification

<enforcement>
Claude Code MUST NOT skip or defer verification. All collection fields in Pydantic models accepting external input MUST use `Field(default_factory=dict)` or `Field(default_factory=list)` to prevent shared mutable state bugs across model instances.
</enforcement>