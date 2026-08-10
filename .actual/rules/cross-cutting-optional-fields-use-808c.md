# Enforce Pydantic BaseModel for Public API Contract Validation: Optional Fields Use

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: Optional fields MUST use Optional type annotation and provide default values or default_factory functions.
- **R-PYDANTIC-002** MUST: All public API endpoint models inherit from Pydantic BaseModel with explicit field types.
- **R-PYDANTIC-003** MUST: Models with numeric constraints enforce bounds using Field parameters (ge, le, etc.).
- **R-PYDANTIC-004** MUST: Datetime fields include Config class with json_encoders mapping datetime to isoformat method.
- **R-PYDANTIC-005** MUST: Mutable defaults (Dict, List) use Field with default_factory rather than direct assignment.
- **R-PYDANTIC-006** SHOULD: Document field-level descriptions using Field description parameter to populate OpenAPI schema.

### Verify

```bash
# Discover the project's dependency manifest and identify the validation library version
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "setup.py" | head -1

# Locate the lock or resolution artifact to confirm exact installed version
find . -name "poetry.lock" -o -name "Pipfile.lock" -o -name "uv.lock" | head -1

# Discover the project's test suite location and execute validation-specific tests
find . -type d -name "tests" -o -name "test" | head -1

# Run type checking to verify all public API models have complete type annotations
which mypy || which pyright || which pyre

# Verify BaseModel inheritance in public API files
grep -r "class.*BaseModel" --include="*.py" | grep -E "(router|endpoint|schema|model)"

# Verify Field constraints are used for numeric bounds
grep -r "Field.*ge=\|Field.*le=" --include="*.py"

# Verify Optional fields have defaults
grep -r "Optional\[" --include="*.py" -A 1 | grep -E "default|default_factory"
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Optional fields consistently use Optional type annotation paired with default values or default_factory functions
- Static type checking passes with no untyped public API model fields
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models must be inspected for BaseModel inheritance, Field constraints, Optional type annotations with defaults, and datetime serialization configuration before code is considered complete.
</enforcement>