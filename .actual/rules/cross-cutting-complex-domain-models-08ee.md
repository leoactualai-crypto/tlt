# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Complex Domain Models

These rules are ALWAYS ACTIVE for all internal API domain models, validation classes, and service contracts within the monorepo that handle photo processing, event management, and experience tracking.

### Rules

- **R-PYDANTIC-001** SHOULD: Complex domain models SHOULD be decomposed into focused validation classes representing distinct contexts (e.g., CloudEventContext, DiscordContext, TimerContext).
- **R-PYDANTIC-002** MUST: All domain validation models MUST inherit from Pydantic BaseModel for internal API contracts.
- **R-PYDANTIC-003** MUST: Numeric score fields MUST use Field constraints (ge, le) to enforce valid ranges (e.g., 0.0-1.0 for quality scores).
- **R-PYDANTIC-004** MUST: All Field definitions MUST include description parameters for self-documentation.
- **R-PYDANTIC-005** SHOULD: Mutable default values SHOULD use Field(default_factory=dict) or Field(default_factory=list) to avoid shared state bugs.
- **R-PYDANTIC-006** SHOULD: Optional fields SHOULD use Optional[T] type hints with default=None.
- **R-PYDANTIC-007** SHOULD: Output models SHOULD include reasoning or metadata fields to support debugging and observability.
- **R-PYDANTIC-008** MUST: Validation behavior MUST be tested with pytest fixtures covering valid, invalid, and edge-case inputs.

### Verify

```bash
# Count BaseModel domain validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Count Field constraint usage
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid
- Unit tests validate constraint enforcement (e.g., scores outside 0.0-1.0 raise ValidationError)
- Type checking with mypy verifies Optional and type hint correctness

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients. Quarterly audits MUST identify validation models missing description fields.
</enforcement>