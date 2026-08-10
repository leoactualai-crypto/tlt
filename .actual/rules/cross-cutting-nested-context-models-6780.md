# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Nested Context Models

These rules are ALWAYS ACTIVE for all domain models representing events, decisions, API contracts, and workflow states that cross service boundaries or participate in integration testing.

### Rules

- **R-PYDANTIC-001** SHOULD: Nested context models SHOULD be composed using typed references to other validation models rather than untyped dictionaries.
- **R-PYDANTIC-002** MUST: All domain models representing events, decisions, API contracts, and workflow states MUST inherit from the validation base class and declare field constraints.
- **R-PYDANTIC-003** MUST: Agent reasoning decision models with conditional field requirements MUST document the decision type or context that determines which fields are required using field descriptions.
- **R-PYDANTIC-004** MUST: Event context models including CloudEvent, Discord, Timer, and generic event structures MUST use typed validation models.
- **R-PYDANTIC-005** MUST: API request and response models for event management, experience tracking, and task submission endpoints MUST declare response models for automatic schema generation.
- **R-PYDANTIC-006** MUST: Photo analysis and quality assessment output models MUST include confidence scores and reasoning fields to support debugging and observability.
- **R-PYDANTIC-007** MUST: Service status, health check, and metrics response models MUST maintain consistent schemas across multiple service boundaries.
- **R-PYDANTIC-008** MUST: Task lifecycle and provenance tracking state models MUST use validation models for all state transitions.
- **R-PYDANTIC-009** MAY: Internal function parameter validation not crossing service boundaries MAY use simpler validation approaches with team lead approval.
- **R-PYDANTIC-010** MAY: Temporary validation bypass for performance-critical hot paths MAY be granted with architecture review and documented justification.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# 2. Locate and execute the project's type checking verification script
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# 3. Discover the project's test runner configuration and execute integration tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# 4. Discover the project's API documentation generation tool
find . -name 'openapi.json' -o -name 'swagger.json' -o -path '*/docs/*' -type f | head -1

# 5. Verify all domain models inherit from validation base class
grep -r 'class.*BaseModel' --include='*.py' | grep -E '(models|schemas|domain)' | wc -l
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation
- API endpoints declare response models and generate valid OpenAPI schemas automatically
- Static type checking in continuous integration pipeline detects missing model annotations
- Code review checklist requires validation models for all new API endpoints and external event handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain models crossing service boundaries MUST be validated against these rules before code acceptance.
</enforcement>