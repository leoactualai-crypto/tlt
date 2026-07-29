# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Field Descriptions Provided

These rules are ALWAYS ACTIVE for all FastAPI service endpoints, domain validation models, and external API contracts across Discord adapters, MCP services, and monitoring workflows.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoint request/response models SHALL be defined as Pydantic BaseModel subclasses with explicit type annotations.
- **R-PYDANTIC-002** MUST: All FastAPI router decorators (@router.post, @router.get, @router.put, @router.delete) SHALL include the response_model parameter to enforce output validation and enable automatic schema generation.
- **R-PYDANTIC-003** SHOULD: Field descriptions SHOULD be provided using Field(description='...') for models used in external contracts to support API documentation.
- **R-PYDANTIC-004** MUST: Numeric fields with bounded ranges SHALL use Field(ge=X, le=Y) constraints to encode domain rules directly in model definitions.
- **R-PYDANTIC-005** SHOULD: Request models SHOULD be named with 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models SHOULD be named with 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-PYDANTIC-006** MUST: Optional fields SHALL be declared with Optional[T] type annotation and explicit None default value.
- **R-PYDANTIC-007** MUST: All imports of BaseModel and Field SHALL come from pydantic, and Optional/List from typing.

### Verify

```bash
# Count BaseModel subclasses with Create/Response/Output/Update naming pattern
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count FastAPI router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with bounded ranges (ge/le)
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l

# Verify Field descriptions on external contract models
grep -r 'Field(.*description=' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Field descriptions are provided for all models representing external API contracts or workflow state
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, Field constraints, and Field descriptions
- No manual validation logic using isinstance checks or conditional logic exists within endpoint handlers for fields covered by model constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints and domain models must pass the verify commands before acceptance. CI pipeline MUST fail if new router endpoints lack response_model parameter or if domain models lack Field constraints for bounded numeric values.
</enforcement>