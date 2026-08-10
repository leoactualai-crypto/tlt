# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Field Constraints Including

These rules are ALWAYS ACTIVE for all domain models representing events, decisions, API contracts, and workflow states that cross service boundaries or participate in integration testing.

### Rules

- **R-PYDANTIC-001** MUST: Field constraints including numeric bounds, string patterns, required vs optional semantics, and default values MUST be declared using field descriptor annotations.
- **R-PYDANTIC-002** MUST: All domain models representing events, decisions, API contracts, and workflow states MUST inherit from the validation base class (Pydantic BaseModel).
- **R-PYDANTIC-003** MUST: Models with conditional field requirements MUST document the decision type or context that determines which fields are required using field descriptions.
- **R-PYDANTIC-004** SHOULD: Define models in dedicated modules separate from business logic to enable reuse across service boundaries and facilitate schema evolution.
- **R-PYDANTIC-005** SHOULD: Include confidence scores and reasoning fields in decision and analysis output models to support debugging and observability in production.
- **R-PYDANTIC-006** MAY: Temporary validation bypass for performance-critical hot paths may be used only with architecture review and documented justification.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

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
- Code review checklist confirms validation models for all new API endpoints and external event handlers.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing unvalidated domain models crossing service boundaries are rejected. Runtime validation failures trigger error logging with model schema and invalid data. API endpoints without declared response models fail OpenAPI schema generation checks in CI.
</enforcement>