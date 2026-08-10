# Enforce Pydantic BaseModel for Public API Contract Validation: Numeric Scores Ratings

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: Numeric scores and ratings MUST use Field constraints with `ge` and `le` parameters to enforce valid ranges.
- **R-PYDANTIC-002** MUST: All public API endpoint request and response models MUST inherit from Pydantic BaseModel with explicit field types.
- **R-PYDANTIC-003** MUST: Datetime fields MUST include a Config class with json_encoders mapping datetime to isoformat method for consistent serialization.
- **R-PYDANTIC-004** MUST: Mutable defaults (Dict, List) MUST use Field with default_factory rather than direct assignment to avoid shared mutable state.
- **R-PYDANTIC-005** SHOULD: Score and rating fields SHOULD combine Field constraints with enum types to enforce both numeric bounds and categorical validity.
- **R-PYDANTIC-006** SHOULD: Field-level descriptions SHOULD be documented using Field description parameter to populate OpenAPI schema documentation.

### Verify

```bash
# Discover the project's dependency manifest and identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Locate the lock or resolution artifact to confirm exact installed version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Discover the project's test suite location and execute validation-specific tests
find . -type d -name 'tests' -o -name 'test' | head -1

# Discover the project's static analysis configuration and run type checking
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' | head -1

# Verify all public API models inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E "(router|endpoint|schema|model)" | wc -l

# Verify Field constraints on numeric fields
grep -r "Field.*ge.*le" --include="*.py" | wc -l
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters with `ge` and `le` and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Static type checking in CI pipeline verifies all public API models have complete type annotations
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses
- Code review checklist includes verification that new API models follow BaseModel patterns with Field constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models must be validated against these rules before acceptance. Pull requests introducing public API models without BaseModel inheritance are blocked by automated checks.
</enforcement>