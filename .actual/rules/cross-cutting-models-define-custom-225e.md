# Enforce Pydantic BaseModel for Public API Contract Validation: Models Define Custom

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: All public API endpoint request and response models inherit from Pydantic BaseModel with explicit field types and validation.
- **R-PYDANTIC-002** MUST: Models with numeric constraints (scores, ratings, percentages) enforce bounds using Field parameters with ge/le constraints and reject out-of-range values during instantiation.
- **R-PYDANTIC-003** MUST: Datetime fields include Config class with json_encoders mapping datetime to isoformat method to ensure consistent serialization across all endpoints.
- **R-PYDANTIC-004** MUST: Mutable default fields (Dict, List) use Field with default_factory rather than direct assignment to avoid shared mutable state across model instances.
- **R-PYDANTIC-005** MAY: Models MAY define custom validators using validator decorators for cross-field validation logic.
- **R-PYDANTIC-006** SHOULD: Field-level descriptions use Field description parameter to populate OpenAPI schema documentation automatically.
- **R-PYDANTIC-007** SHOULD: Score and rating fields combine Field constraints with enum types to enforce both numeric bounds and categorical validity.

### Verify

```bash
# Discover the project's dependency manifest and identify the validation library version
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'setup.py' -o -name 'Pipfile' | head -1

# Locate the lock or resolution artifact to confirm the exact installed version
find . -name 'requirements.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Discover the project's test suite location and execute validation-specific test cases
find . -type d -name 'tests' -o -name 'test' | head -1

# Discover the project's static analysis configuration and run type checking
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | head -1
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages.
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation.
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code.
- Mutable default fields use default_factory to prevent shared state across instances.
- Static type checking in CI pipeline verifies all public API models have complete type annotations.
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses.

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models must be validated against these rules before acceptance.
</enforcement>