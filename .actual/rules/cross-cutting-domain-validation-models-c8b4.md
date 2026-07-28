# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Domain Validation Models

These rules are ALWAYS ACTIVE for all domain validation models in internal API contracts across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** MUST: All domain validation models for internal API contracts MUST inherit from Pydantic BaseModel
- **R-PYDANTIC-002** MUST: Numeric score fields MUST use Field constraints (ge, le) to enforce valid ranges
- **R-PYDANTIC-003** MUST: All Field definitions MUST include description parameters for self-documentation
- **R-PYDANTIC-004** SHOULD: Use Field(default_factory=dict) or Field(default_factory=list) for mutable default values to avoid shared state bugs
- **R-PYDANTIC-005** SHOULD: Decompose complex models into focused context classes for reusability
- **R-PYDANTIC-006** SHOULD: Include reasoning or metadata fields in output models to support debugging and observability
- **R-PYDANTIC-007** SHOULD: Use Optional[T] type hints with default=None for optional fields

### Verify

```bash
# Verify BaseModel inheritance across domain validation models
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Verify Field constraints on numeric fields
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid
- Code review verifies BaseModel inheritance in new domain models
- CI linting rules verify Field constraints on numeric score fields
- Unit tests validate constraint enforcement (e.g., scores outside 0.0-1.0 raise ValidationError)
- Type checking with mypy verifies Optional and type hint correctness

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients. Quarterly audits MUST identify validation models missing description fields.
</enforcement>