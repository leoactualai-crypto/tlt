# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Field Definitions Include

These rules are ALWAYS ACTIVE for all domain validation models in internal API contracts across photo processing, event management, and experience tracking services.

### Rules

- **R-PYDANTIC-001** MUST: All Field definitions MUST include a description parameter documenting the field's purpose and valid values.

### Verify

```bash
# Verify BaseModel domain validation models exist
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l

# Verify Field constraints are used for numeric fields
grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l

# Verify validation models parse successfully
python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"

# Verify Field definitions include description parameters
grep -r 'Field(' --include='*.py' monorepo/tlt/ | grep -v 'description=' | wc -l
```

**Accept when:**
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if domain models lack BaseModel inheritance or Field constraints. CI MUST fail if validation tests do not cover constraint violations. Quarterly audits MUST identify validation models missing description fields.
</enforcement>