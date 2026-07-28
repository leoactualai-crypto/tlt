# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Optional Fields Use

These rules are ALWAYS ACTIVE for all internal API domain validation models across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** SHOULD: Optional fields SHOULD use Optional type hints and default values via Field(default_factory=...) for mutable defaults
- **R-PYDANTIC-002** MUST: All domain validation models inheriting from Pydantic BaseModel MUST include Field constraints (ge, le, description) for numeric score fields enforcing valid ranges (e.g., 0.0-1.0)
- **R-PYDANTIC-003** MUST: All Field definitions MUST include description parameters for self-documentation
- **R-PYDANTIC-004** SHOULD: Complex models SHOULD be decomposed into focused context classes (e.g., separate DiscordContext from EventContext) for reusability
- **R-PYDANTIC-005** SHOULD: Output models SHOULD include reasoning or metadata fields to support debugging and observability
- **R-PYDANTIC-006** MUST: Mutable default values MUST use Field(default_factory=dict) or Field(default_factory=list) to avoid shared state bugs

### Verify

```bash
# Count BaseModel domain validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Count Field constraints on numeric fields
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid
- Optional fields use Optional type hints with default=None or Field(default_factory=...)
- Mutable defaults use Field(default_factory=dict) or Field(default_factory=list)

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Type checking with mypy MUST verify Optional and type hint correctness. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients.
</enforcement>