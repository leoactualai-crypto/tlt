# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Public Contracts Process

These rules are ALWAYS ACTIVE for all internal API contracts, domain models, and public service functions across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** MUST: Public API contracts (process_*, create_*, get_*) MUST accept and return validated BaseModel instances.
- **R-PYDANTIC-002** MUST: All domain validation models MUST inherit from Pydantic BaseModel.
- **R-PYDANTIC-003** MUST: Numeric score fields MUST use Field constraints (ge, le) to enforce valid ranges (e.g., 0.0-1.0).
- **R-PYDANTIC-004** MUST: All Field definitions MUST include description parameters for self-documentation.
- **R-PYDANTIC-005** MUST: Mutable default values MUST use Field(default_factory=dict) or Field(default_factory=list) to avoid shared state bugs.
- **R-PYDANTIC-006** SHOULD: Complex models SHOULD be decomposed into focused context classes (e.g., separate DiscordContext from EventContext) for reusability.
- **R-PYDANTIC-007** SHOULD: Output models SHOULD include reasoning or metadata fields to support debugging and observability.
- **R-PYDANTIC-008** SHOULD: Optional fields SHOULD use Optional[T] type hints with default=None.

### Verify

```bash
# Count BaseModel domain validation models in scope
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Count Field constraint usage for numeric validation
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"

# Verify Field descriptions are present on numeric constraints
grep -r "Field(ge=.*description=" --include='*.py' monorepo/tlt/ | wc -l
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid
- Public API contracts (process_*, create_*, get_*) accept and return BaseModel instances
- Unit tests validate constraint enforcement (e.g., scores outside 0.0-1.0 raise ValidationError)
- Type checking with mypy verifies Optional and type hint correctness

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients. Quarterly audits MUST identify validation models missing description fields.
</enforcement>