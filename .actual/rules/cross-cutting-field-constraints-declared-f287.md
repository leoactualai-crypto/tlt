# Enforce Pydantic BaseModel for Public API Contract Validation: Field Constraints Declared

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: Field constraints MUST be declared using Field descriptors with validation parameters for numeric bounds, string patterns, and collection sizes.
- **R-PYDANTIC-002** MUST: All public API endpoint request and response models inherit from Pydantic BaseModel with explicit field types.
- **R-PYDANTIC-003** MUST: Mutable default values (Dict, List) use default_factory rather than direct assignment to prevent shared mutable state.
- **R-PYDANTIC-004** MUST: Datetime fields include Config class with json_encoders mapping datetime to isoformat method for consistent serialization.
- **R-PYDANTIC-005** SHOULD: Score and rating fields combine Field constraints with enum types to enforce both numeric bounds and categorical validity.
- **R-PYDANTIC-006** SHOULD: Field-level descriptions use Field description parameter to populate OpenAPI schema documentation automatically.

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
grep -r "class.*BaseModel" --include="*.py" | grep -E "(router|endpoint|schema|model)"

# Verify Field constraints are declared
grep -r "Field(" --include="*.py" | grep -E "(ge=|le=|min_length|max_length|pattern)"

# Verify datetime serialization configuration
grep -r "json_encoders" --include="*.py" | grep -i datetime
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Mutable default values use default_factory pattern across all models
- Field descriptions are present in OpenAPI schema documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Static type checking in continuous integration pipeline MUST verify all public API models have complete type annotations. Integration tests MUST validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses. Code review checklist MUST include verification that new API models follow BaseModel patterns with Field constraints.
</enforcement>