# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Validation Models Include

These rules are ALWAYS ACTIVE for all domain validation models in internal API contracts across photo processing, event management, and experience tracking services.

### Rules

- **R-VAL-001** MUST: All domain validation models MUST inherit from `pydantic.BaseModel` for input/output validation across internal API service boundaries.
- **R-VAL-002** MUST: Numeric score and quality fields MUST use `Field(ge=..., le=..., description='...')` constraints to enforce valid ranges (e.g., 0.0–1.0 for quality scores).
- **R-VAL-003** MUST: All `Field` definitions MUST include `description` parameters for self-documentation of domain invariants.
- **R-VAL-004** SHOULD: Validation models SHOULD include reasoning or metadata fields to support observability and debugging of validation decisions.
- **R-VAL-005** MUST: Mutable default values in models MUST use `Field(default_factory=dict)` or `Field(default_factory=list)` to avoid shared state bugs.
- **R-VAL-006** MUST: Optional fields MUST use `Optional[T]` type hints with `default=None` for clarity.
- **R-VAL-007** SHOULD: Complex models SHOULD decompose into focused context classes (e.g., separate `DiscordContext` from `EventContext`) for reusability across event trigger types.

### Verify

```bash
# Count BaseModel domain validation models in scope
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Count Field constraint usage on numeric fields
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
- Validation tests cover constraint violations and raise ValidationError appropriately
- Type checking with mypy verifies Optional and type hint correctness

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Runtime ValidationError exceptions MUST be logged and returned as 400 Bad Request to API clients. Quarterly audits MUST identify validation models missing description fields.
</enforcement>