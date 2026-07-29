# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Domain Models Include

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain validation models for Discord adapter services, MCP service domain models, service monitoring response models, and any model representing external API contracts or workflow state.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified on router decorators (@router.post, @router.get, @router.put, @router.delete).
- **R-PYDANTIC-002** MUST: Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models.
- **R-PYDANTIC-003** MUST: Import BaseModel and Field from pydantic, and Optional/List from typing for all domain validation models.
- **R-PYDANTIC-004** SHOULD: Name request models with 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models with 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-PYDANTIC-005** SHOULD: Use Field(description='...') for API documentation on all model fields.
- **R-PYDANTIC-006** MAY: Domain models MAY include nested Pydantic models or List types for complex structures (e.g., photos: Optional[List[str]]).
- **R-PYDANTIC-007** SHOULD: Apply response_model parameter to all FastAPI router decorators to enforce output validation and enable automatic schema generation.
- **R-PYDANTIC-008** SHOULD: Test models independently by instantiating with valid and invalid data to verify Field constraints trigger ValidationError as expected.

### Verify

```bash
# Count BaseModel subclasses with naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with numeric bounds
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l

# Verify all BaseModel subclasses have type annotations
mypy monorepo/tlt --strict --no-implicit-optional
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints
- mypy static analysis passes with strict type checking on all model fields

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model parameter. Code review MUST block merge if domain models lack Field constraints for bounded numeric values. Linting warnings MUST be issued for BaseModel subclasses without type annotations.
</enforcement>