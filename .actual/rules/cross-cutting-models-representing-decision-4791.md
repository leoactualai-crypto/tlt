# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Models Representing Decision

These rules are ALWAYS ACTIVE for all domain models representing events, decisions, API contracts, workflow states, and data crossing service boundaries.

### Rules

- **R-PYDANTIC-001** SHOULD: Models representing decision outputs or analysis results SHOULD include confidence scores, reasoning fields, and metadata dictionaries to support observability.
- **R-PYDANTIC-002** MUST: All domain models representing events, decisions, API contracts, and workflow states crossing service boundaries MUST inherit from the validation base class and declare field constraints.
- **R-PYDANTIC-003** SHOULD: Define models in dedicated modules separate from business logic to enable reuse across service boundaries and facilitate schema evolution.
- **R-PYDANTIC-004** SHOULD: For models with conditional field requirements, document the decision type or context that determines which fields are required using field descriptions.
- **R-PYDANTIC-005** MUST: API endpoints MUST declare response models to enable automatic OpenAPI schema generation.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# Execute the project's type checking verification script
# (Discover from project configuration)

# Discover the project's test runner configuration and execute integration tests
# that validate model serialization, constraint enforcement, and API contract compliance

# Discover the project's API documentation generation tool and verify
# that all endpoint response models produce valid schema definitions
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints.
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation.
- API endpoints declare response models and generate valid OpenAPI schemas automatically.
- Static type checking in continuous integration pipeline detects missing model annotations.
- Code review checklist requires validation models for all new API endpoints and external event handlers.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new domain models crossing service boundaries MUST be validated against these rules before acceptance.
</enforcement>