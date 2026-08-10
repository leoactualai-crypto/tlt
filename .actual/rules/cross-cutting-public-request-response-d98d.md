# Enforce Pydantic BaseModel for Public API Contract Validation: Public Request Response

These rules are ALWAYS ACTIVE for all public API request and response models, including FastAPI router endpoints, MCP service tool schemas, agent reasoning models, CloudEvent payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: All public API request and response models MUST inherit from BaseModel and define explicit field types with type annotations.
- **R-PYDANTIC-002** MUST: Models with numeric constraints (scores, ratings) MUST enforce bounds using Field parameters with ge/le constraints and reject out-of-range values during instantiation.
- **R-PYDANTIC-003** MUST: Datetime fields MUST include a Config class with json_encoders mapping datetime to isoformat method to ensure consistent serialization across all endpoints.
- **R-PYDANTIC-004** MUST: Mutable default values (Dict, List) MUST use Field with default_factory rather than direct assignment to avoid shared mutable state across model instances.
- **R-PYDANTIC-005** SHOULD: Field-level descriptions SHOULD be documented using Field description parameter to populate OpenAPI schema documentation automatically.

### Verify

```bash
# 1. Discover the project's dependency manifest and identify the validation library version
# Locate the lock or resolution artifact to confirm the exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# 2. Discover the project's test suite location and execute validation-specific test cases
# Verify Field constraints, type checking, and serialization behavior
find . -path '*/test*' -name '*test*.py' -type f | grep -E '(validation|model|schema)' | head -5

# 3. Discover the project's static analysis configuration and run type checking
# Verify all public API models have complete type annotations
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.pylintrc' | head -1

# 4. Verify all public API endpoint models inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E '(request|response|schema)' | wc -l

# 5. Verify Field constraints are applied to numeric fields
grep -r "Field.*ge=\|Field.*le=" --include="*.py" | wc -l
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Mutable default values use default_factory pattern consistently across all models
- Static type checking in CI pipeline verifies all public API models have complete type annotations
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models must be inspected for BaseModel inheritance and Field constraint compliance before code is committed.
</enforcement>