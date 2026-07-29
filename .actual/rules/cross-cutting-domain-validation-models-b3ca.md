# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Domain Validation Models

These rules are ALWAYS ACTIVE for all FastAPI service endpoints, domain validation models, and API contracts across Discord adapters, MCP services, and service monitoring components.

### Rules

- **R-DOMAIN-001** MUST: All domain validation models MUST inherit from Pydantic BaseModel with explicit type annotations for each field.
- **R-DOMAIN-002** MUST: All FastAPI router endpoints (@router.post, @router.get, @router.put, @router.delete) MUST specify a response_model parameter to enforce output validation and enable automatic schema generation.
- **R-DOMAIN-003** MUST: Numeric fields with bounded ranges MUST use Field(ge=X, le=Y) constraints to encode domain rules directly in model definitions.
- **R-DOMAIN-004** SHOULD: Request models SHOULD be named with 'Create' or 'Update' suffix (e.g., ReminderCreate, ReminderUpdate) and response models with 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-DOMAIN-005** SHOULD: Optional fields SHOULD use Optional[T] = None syntax with Field(description='...') for API documentation.
- **R-DOMAIN-006** MAY: Complex validation logic requiring cross-field dependencies or external data MAY use custom validators, but MUST document the rationale in code comments.

### Verify

```bash
# Count BaseModel subclasses with naming conventions
grep -r 'class.*BaseModel' . --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' . --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with bounded ranges
grep -r 'Field(.*ge=.*le=' . --include='*.py' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints
- New domain models follow naming conventions (Create/Update/Response suffixes) and include type annotations on all fields

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model parameter. Code review MUST block merge if domain models lack Field constraints for bounded numeric values. Linting warnings MUST be issued for BaseModel subclasses without type annotations.
</enforcement>