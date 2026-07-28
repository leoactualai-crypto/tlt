# Enforce Pydantic BaseModel Validation for All Input Data Models: Business Logic Not

These rules are ALWAYS ACTIVE for all services, agents, and adapters that process external input, API requests, event payloads, or user-submitted data. All data models accepting untrusted input MUST use Pydantic BaseModel validation.

### Rules

- **R-PYDANTIC-001** MUST NOT: Business logic MUST NOT accept raw dictionaries or unvalidated JSON for external input; validation MUST occur at deserialization boundaries.
- **R-PYDANTIC-002** MUST: All MCP service models (gateway, RSVP, event_manager, photo_vibe_check, guild_manager) MUST inherit from Pydantic BaseModel for external input handling.
- **R-PYDANTIC-003** MUST: All agent state models (ambient_event_agent state, AgentTask, IncomingEvent) MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-004** MUST: All adapter models (discord_adapter experience manager) MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-005** MUST: API request/response models for HTTP endpoints MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-006** MUST: CloudEvent and timer context models MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-007** MUST: Configuration models loaded from external sources MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-008** MUST: Numeric fields with semantic constraints (scores, ratings, percentages) MUST use Field validators with ge/le bounds (e.g., Field(ge=0.0, le=1.0) for normalized scores, Field(ge=1, le=5) for rating scales).
- **R-PYDANTIC-009** MUST: Collection fields (Dict, List) MUST consistently use Field(default_factory) to prevent mutable default bugs.
- **R-PYDANTIC-010** MUST: Timestamp fields MUST use timezone-aware datetime with default_factory for UTC timestamp generation (e.g., Field(default_factory=lambda: datetime.now(timezone.utc))).
- **R-PYDANTIC-011** SHOULD: Define nested context models (DiscordContext, TimerContext, EventContext) as separate BaseModel classes for reusability and clear validation boundaries.

### Verify

```bash
# Count BaseModel classes in model files
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
- No raw dictionaries or unvalidated JSON are accepted at API boundaries or event handlers

<enforcement>
Clause Code MUST NOT skip or defer verification. All new models accepting external input MUST inherit from Pydantic BaseModel. CI pipeline MUST fail if validation constraints are missing for numeric fields with semantic bounds. Code review MUST block merge if validation is bypassed at trust boundaries.
</enforcement>