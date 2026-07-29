# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Request Response Models

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain validation models for Discord adapter services, MCP service domain models, and service monitoring response models.

### Rules

- **R-PYDANTIC-001** MUST: Request and response models for FastAPI router endpoints MUST be defined as Pydantic BaseModel subclasses with descriptive class names (e.g., ReminderCreate, ExperienceResponse, PhotoAnalysisOutput).
- **R-PYDANTIC-002** MUST: All FastAPI router decorators (@router.post, @router.get, @router.put, @router.delete) MUST specify the response_model parameter to enforce output validation and enable automatic schema generation.
- **R-PYDANTIC-003** MUST: Numeric fields with bounded ranges MUST use Field(ge=X, le=Y) constraints to encode domain rules directly in model definitions.
- **R-PYDANTIC-004** SHOULD: Name request models with 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models with 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-PYDANTIC-005** SHOULD: Use Field(description='...') for all model fields to enable accurate API documentation generation.
- **R-PYDANTIC-006** SHOULD: Mark optional fields with Optional[T] = None type annotation and default value.
- **R-PYDANTIC-007** SHOULD: Import BaseModel and Field from pydantic, and Optional/List from typing for all domain validation models.
- **R-PYDANTIC-008** MAY: Use custom validators for complex validation logic requiring cross-field dependencies or external data validation.

### Verify

```bash
# Count BaseModel subclasses with descriptive naming patterns
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with numeric bounds
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints
- Code review checklist confirms Pydantic BaseModel adoption for new API endpoints
- Static analysis with mypy enforces type annotations on model fields

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model parameter. Code review MUST block merge if domain models lack Field constraints for bounded numeric values.
</enforcement>