# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Optional Fields Use

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain validation models for Discord adapter services, MCP service domain models, and service monitoring response models that represent external API contracts or workflow state.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified on router decorators (@router.post, @router.get, @router.put, @router.delete).
- **R-PYDANTIC-002** SHOULD: Optional fields SHOULD use typing.Optional with default values to distinguish required from optional parameters.
- **R-PYDANTIC-003** MUST: Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models.
- **R-PYDANTIC-004** MUST: Request models use 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models use 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-PYDANTIC-005** MUST: Import BaseModel and Field from pydantic, and Optional/List from typing for all domain validation models.
- **R-PYDANTIC-006** SHOULD: Use Field(description='...') for API documentation on model fields.
- **R-PYDANTIC-007** MUST: All model fields include type annotations.

### Verify

```bash
# Count BaseModel subclasses with naming convention
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with numeric bounds
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l

# Verify all BaseModel subclasses have type annotations
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 5 | grep -E ': (str|int|float|bool|Optional|List)' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Request models follow 'Create'/'Update' naming convention and response models use 'Response' suffix
- All model fields include explicit type annotations
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, Field constraints, and type annotations
- Optional fields explicitly use typing.Optional with default values (e.g., field: Optional[str] = None)

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model parameter. Code review MUST block merge if domain models lack Field constraints for bounded numeric values. Linting warnings MUST be issued for BaseModel subclasses without type annotations.
</enforcement>