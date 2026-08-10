# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Integration Test Assertions

These rules are ALWAYS ACTIVE for all domain models, event handlers, API endpoints, and integration tests that process external events, validate agent decisions, or cross service boundaries.

### Rules

- **R-PYDANTIC-001** MUST: Integration test assertions MUST validate model instances against expected field values, constraint satisfaction, and serialization round-trip correctness.
- **R-PYDANTIC-002** MUST: All domain models representing events, decisions, API contracts, and workflow states MUST inherit from the validation base class and declare field constraints.
- **R-PYDANTIC-003** MUST: Agent reasoning decision models MUST enforce conditional field requirements based on decision type using framework-level validation support.
- **R-PYDANTIC-004** MUST: Event context models including CloudEvent, Discord, Timer, and generic event structures MUST use BaseModel inheritance with Field descriptors for constraint declaration.
- **R-PYDANTIC-005** MUST: API request and response models for event management, experience tracking, and task submission endpoints MUST declare response models and generate valid OpenAPI schemas automatically.
- **R-PYDANTIC-006** MUST: Photo analysis and quality assessment output models MUST include scored ratings with constraint validation.
- **R-PYDANTIC-007** MUST: Service status, health check, and metrics response models MUST maintain consistent schemas across multiple service boundaries.
- **R-PYDANTIC-008** MUST: Task lifecycle and provenance tracking state models MUST validate state transitions through model constraints.
- **R-PYDANTIC-009** SHOULD: Define models in dedicated modules separate from business logic to enable reuse across service boundaries and facilitate schema evolution.
- **R-PYDANTIC-010** SHOULD: For models with conditional field requirements, document the decision type or context that determines which fields are required using field descriptions.
- **R-PYDANTIC-011** SHOULD: Include confidence scores and reasoning fields in decision and analysis output models to support debugging and observability in production.
- **R-PYDANTIC-012** MAY: Temporary validation bypass for performance-critical hot paths requires architecture review and documented justification.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# Locate and execute the project's type checking verification script
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# Discover the project's test runner configuration and execute integration tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# Discover the project's API documentation generation tool and verify schema definitions
find . -name 'openapi.json' -o -name 'openapi.yaml' -o -name 'swagger.json' 2>/dev/null | head -1
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation
- API endpoints declare response models and generate valid OpenAPI schemas automatically
- Static type checking in continuous integration pipeline detects missing model annotations
- Integration test suite validates model constraint enforcement and serialization correctness
- Code review checklist requires validation models for all new API endpoints and external event handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain models crossing service boundaries MUST use BaseModel validation. Pull requests introducing unvalidated domain models are rejected in code review. Runtime validation failures trigger error logging with model schema and invalid data for debugging.
</enforcement>