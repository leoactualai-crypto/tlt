# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Fastapi Router Decorators

These rules are ALWAYS ACTIVE for all FastAPI service endpoints, domain validation models, and API contract definitions across Discord adapters, MCP services, and service monitoring components.

### Rules

- **R-PYDANTIC-001** MUST: FastAPI router decorators MUST specify `response_model` parameter using the corresponding Pydantic BaseModel to enforce contract validation (e.g., `@router.post('/', response_model=ReminderResponse)`).
- **R-PYDANTIC-002** MUST: All domain validation models representing external API contracts or workflow state MUST inherit from `pydantic.BaseModel`.
- **R-PYDANTIC-003** MUST: Numeric fields with bounded ranges MUST use `Field(ge=X, le=Y)` constraints to encode domain rules directly in model definitions.
- **R-PYDANTIC-004** SHOULD: Request models SHOULD use 'Create' or 'Update' suffix (e.g., `ReminderCreate`) and response models SHOULD use 'Response' suffix (e.g., `ReminderResponse`) for clarity.
- **R-PYDANTIC-005** SHOULD: Optional fields SHOULD be declared with `Optional[T] = None` type annotation and default value.
- **R-PYDANTIC-006** SHOULD: Field definitions SHOULD include `description` parameter for API documentation generation.
- **R-PYDANTIC-007** MUST: All model fields MUST have explicit type annotations for IDE autocomplete and static type checking.

### Verify

```bash
# Count BaseModel subclasses with naming conventions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router decorators with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with numeric bounds
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l

# Verify no router endpoints lack response_model
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep -v 'response_model=' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with `response_model` parameter specified.
- Numeric fields with bounded ranges use `Field(ge=X, le=Y)` constraints consistently across domain models.
- Request models follow 'Create'/'Update' naming convention and response models follow 'Response' naming convention.
- All model fields have explicit type annotations.
- Grep verification commands return expected counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints.
- No router decorators exist without response_model parameter (final grep returns 0).

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints and domain models MUST satisfy R-PYDANTIC-001 through R-PYDANTIC-007 before code review approval. CI pipeline MUST fail if router endpoints lack response_model parameter or if domain models lack required Field constraints.
</enforcement>