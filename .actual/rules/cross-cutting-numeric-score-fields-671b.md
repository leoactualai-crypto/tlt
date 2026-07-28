# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Numeric Score Fields

These rules are ALWAYS ACTIVE for all internal API domain models, validation contracts, and state management structures across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** MUST: Numeric score fields MUST use Field constraints with `ge` (greater-or-equal) and `le` (less-or-equal) to enforce valid ranges (e.g., `ge=0.0, le=1.0`).
- **R-PYDANTIC-002** MUST: All Field definitions on numeric score fields MUST include a `description` parameter for self-documentation.
- **R-PYDANTIC-003** MUST: Domain validation models MUST inherit from Pydantic `BaseModel` for internal API contracts.
- **R-PYDANTIC-004** SHOULD: Decompose complex models into focused context classes (e.g., separate `DiscordContext` from `EventContext`) for reusability.
- **R-PYDANTIC-005** SHOULD: Use `Field(default_factory=dict)` or `Field(default_factory=list)` for mutable default values to avoid shared state bugs.
- **R-PYDANTIC-006** SHOULD: Include reasoning or metadata fields in output models to support debugging and observability.
- **R-PYDANTIC-007** SHOULD: Use `Optional[T]` type hints with `default=None` for optional fields.

### Verify

```bash
# Count BaseModel domain validation models across internal API services
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Count Field constraints with ge/le on numeric score fields
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (`ge`, `le`) to enforce valid ranges
- All Field definitions on numeric score fields include `description` parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid
- Unit tests validate constraint enforcement (e.g., scores outside 0.0–1.0 raise `ValidationError`)
- Type checking with mypy verifies `Optional` and type hint correctness

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints on numeric score fields. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients.
</enforcement>