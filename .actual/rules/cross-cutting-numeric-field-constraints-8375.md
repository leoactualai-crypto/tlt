# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Numeric Field Constraints

These rules are ALWAYS ACTIVE for all FastAPI service endpoints, domain validation models, and API contract definitions across Discord adapters, MCP services, and service monitoring components.

### Rules

- **R-PYDANTIC-001** MUST: Numeric field constraints MUST use Pydantic Field with ge (greater than or equal) and le (less than or equal) parameters where bounds are required (e.g., `quality_score: float = Field(ge=0.0, le=1.0)`).
- **R-PYDANTIC-002** MUST: All FastAPI router endpoint request/response models MUST be defined as Pydantic BaseModel subclasses.
- **R-PYDANTIC-003** MUST: All FastAPI router decorators (@router.post, @router.get, @router.put, @router.delete) MUST specify the response_model parameter to enforce output validation and enable automatic schema generation.
- **R-PYDANTIC-004** SHOULD: Request models SHOULD be named with 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models SHOULD be named with 'Response' suffix (e.g., ReminderResponse) for clarity.
- **R-PYDANTIC-005** SHOULD: Optional fields SHOULD use Optional[T] = None syntax with Field(description='...') for API documentation.
- **R-PYDANTIC-006** MUST: Domain validation models for Discord adapter services (reminder.py, experience_manager.py, rsvp.py), MCP service models (photo_processor.py PhotoAnalysisOutput, GenZVibeCheckOutput), and service monitoring response models (monitor.py TaskStatusResponse, ServiceStatusResponse) MUST use Pydantic BaseModel with appropriate Field constraints.

### Verify

```bash
# Count BaseModel subclasses with Create/Response/Output/Update naming
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l

# Count router endpoints with response_model parameter
grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l

# Count Field constraints with ge and le bounds
grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l
```

**Accept when:**
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints
- All new domain models expose type annotations on model fields verified by mypy static analysis

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model parameter. Code review MUST block merge if domain models lack Field constraints for bounded numeric values. Linting warnings MUST be issued for BaseModel subclasses without type annotations.
</enforcement>