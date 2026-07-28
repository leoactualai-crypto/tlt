# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Services Extend Basemodel

These rules are ALWAYS ACTIVE for all domain validation models, internal API contracts, and service input/output validation across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** MUST: All domain validation models (Output, Context, Create, Response, Event classes) inherit from Pydantic BaseModel.
- **R-PYDANTIC-002** MUST: Numeric score fields use Field constraints (ge, le) to enforce valid ranges (e.g., 0.0–1.0 for quality scores).
- **R-PYDANTIC-003** MUST: All Field definitions include description parameters for self-documentation.
- **R-PYDANTIC-004** SHOULD: Services MAY extend BaseModel with custom validators for cross-field validation logic.
- **R-PYDANTIC-005** MUST: Use Field(default_factory=dict) or Field(default_factory=list) for mutable default values to avoid shared state bugs.
- **R-PYDANTIC-006** SHOULD: Decompose complex models into focused context classes (e.g., separate DiscordContext from EventContext) for reusability.
- **R-PYDANTIC-007** SHOULD: Include reasoning or metadata fields in output models to support debugging and observability.
- **R-PYDANTIC-008** SHOULD: Use Optional[T] type hints with default=None for optional fields.
- **R-PYDANTIC-009** MUST: Test validation behavior with pytest fixtures covering valid, invalid, and edge-case inputs.

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

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients. Quarterly audits MUST identify validation models missing description fields.
</enforcement>