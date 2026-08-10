# Enforce Pydantic BaseModel for Public API Contract Validation: Complex Nested Structures

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent data payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: All public API endpoint request and response models inherit from Pydantic BaseModel with explicit field types and validation.
- **R-PYDANTIC-002** SHOULD: Complex nested structures use default_factory with dict or list constructors rather than mutable default arguments.
- **R-PYDANTIC-003** MUST: Numeric constraint fields (scores, ratings) combine Field constraints (ge, le) with enum types to enforce both numeric bounds and categorical validity.
- **R-PYDANTIC-004** MUST: Datetime fields include Config class with json_encoders mapping datetime to isoformat method to ensure consistent serialization across all endpoints.
- **R-PYDANTIC-005** SHOULD: Field-level descriptions use Field description parameter to populate OpenAPI schema documentation automatically.
- **R-PYDANTIC-006** MUST: MCP service tool request and response schemas inherit from BaseModel with complete type annotations.
- **R-PYDANTIC-007** MUST: Agent reasoning decision models and state representations use BaseModel for runtime type checking and validation.
- **R-PYDANTIC-008** MUST: CloudEvent data payloads and context structures validate through BaseModel subclasses at service boundaries.

### Verify

```bash
# 1. Discover the project's dependency manifest and identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# 2. Locate the lock or resolution artifact to confirm the exact installed version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# 3. Discover the project's test suite location and execute validation-specific test cases
find . -type d -name 'tests' -o -name 'test' | head -1

# 4. Discover the project's static analysis configuration and run type checking
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' | head -1

# 5. Verify all public API models inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E "(router|endpoint|service|schema)" | wc -l

# 6. Verify Field constraints are applied to numeric fields
grep -r "Field.*ge=\|Field.*le=" --include="*.py" | wc -l

# 7. Verify datetime fields use json_encoders
grep -r "json_encoders" --include="*.py" | wc -l

# 8. Verify default_factory usage for mutable defaults
grep -r "default_factory" --include="*.py" | wc -l
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses
- Static type checking in continuous integration pipeline verifies all public API models have complete type annotations
- MCP service tool schemas and agent reasoning models follow BaseModel patterns with Field constraints
- CloudEvent payloads validate through BaseModel subclasses at service boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models MUST inherit from BaseModel. All numeric constraint fields MUST use Field parameters. All datetime fields MUST include json_encoders configuration. All mutable default fields MUST use default_factory.
</enforcement>