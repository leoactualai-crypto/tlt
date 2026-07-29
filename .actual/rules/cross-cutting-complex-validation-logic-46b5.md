# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Complex Validation Logic

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain entities exposed through HTTP APIs, data transfer objects crossing service boundaries, configuration models requiring validation, and monitoring/health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: Use Pydantic BaseModel for all request and response models in FastAPI endpoints.
- **R-PYDANTIC-002** MUST: Specify `response_model` parameter in FastAPI route decorators using Pydantic models for automatic validation and serialization.
- **R-PYDANTIC-003** MUST: Use `Field()` with appropriate constraints (ge, le, description) for numeric bounds and field documentation in domain models.
- **R-PYDANTIC-004** SHOULD: Implement cross-field validation using Pydantic validators or root_validators for complex validation logic beyond basic type checking.
- **R-PYDANTIC-005** SHOULD: Use datetime fields with Pydantic's automatic ISO 8601 string parsing for temporal data in API models.
- **R-PYDANTIC-006** MAY: Use Pydantic validators or root_validators for cross-field constraints beyond basic type checking in complex validation scenarios.

### Verify

```bash
# Count BaseModel subclasses in codebase
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model usage in adapters and services
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify response_model parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- No unvalidated dictionary parameters are detected in FastAPI routes

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist MUST require Pydantic models for all new API endpoints. Automated linting rules MUST detect FastAPI routes without response_model parameter. CI pipeline MUST run grep checks counting BaseModel usage in modified API files. Violations trigger PR comments requesting Pydantic model addition before merge approval and CI failure on detection of unvalidated dictionary parameters.
</enforcement>