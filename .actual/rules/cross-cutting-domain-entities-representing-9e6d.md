# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Domain Entities Representing

These rules are ALWAYS ACTIVE for all domain entities representing external events, agent decisions, API contracts, and workflow states that cross service boundaries.

### Rules

- **R-DOMAIN-001** MUST: All domain entities representing external events, agent decisions, API contracts, and workflow states MUST be defined as validation model classes inheriting from a base validation framework.
- **R-DOMAIN-002** MUST: Agent reasoning decision models with conditional field requirements based on decision type MUST use framework-level validation support to enforce conditional field requirements.
- **R-DOMAIN-003** MUST: Event context models including CloudEvent, Discord, Timer, and generic event structures MUST be defined as validation models.
- **R-DOMAIN-004** MUST: API request and response models for event management, experience tracking, and task submission endpoints MUST be defined as validation models.
- **R-DOMAIN-005** MUST: Photo analysis and quality assessment output models with scored ratings MUST be defined as validation models.
- **R-DOMAIN-006** MUST: Service status, health check, and metrics response models MUST be defined as validation models.
- **R-DOMAIN-007** MUST: Task lifecycle and provenance tracking state models MUST be defined as validation models.
- **R-DOMAIN-008** SHOULD: Define models in dedicated modules separate from business logic to enable reuse across service boundaries and facilitate schema evolution.
- **R-DOMAIN-009** SHOULD: For models with conditional field requirements, document the decision type or context that determines which fields are required using field descriptions.
- **R-DOMAIN-010** SHOULD: Include confidence scores and reasoning fields in decision and analysis output models to support debugging and observability in production.
- **R-DOMAIN-011** MAY: Internal function parameter validation not crossing service boundaries may use simpler validation approaches.
- **R-DOMAIN-012** MAY: Temporary data structures used within single-function scope may use simpler validation approaches.
- **R-DOMAIN-013** MAY: Configuration objects loaded from environment variables or files may use simpler validation approaches.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# Locate and execute the project's type checking verification script
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# Discover the project's test runner configuration and execute integration tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# Discover the project's API documentation generation tool
find . -name 'openapi.json' -o -name 'swagger.json' -o -name '.openapi-generator' 2>/dev/null | head -1
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints.
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation.
- API endpoints declare response models and generate valid OpenAPI schemas automatically.
- Static type checking in continuous integration pipeline detects missing model annotations.
- Code review checklist requires validation models for all new API endpoints and external event handlers.

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain entities crossing service boundaries MUST be validated models. Pull requests introducing unvalidated domain models are rejected in code review.
</enforcement>