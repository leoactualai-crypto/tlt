# Enforce Pydantic BaseModel for Public API Contract Validation: Models Include Field

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent data payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: All public API endpoint request and response models inherit from Pydantic BaseModel with explicit field types and complete type annotations.
- **R-PYDANTIC-002** SHOULD: Models SHOULD include Field descriptions documenting the purpose and constraints of each field for API documentation generation.
- **R-PYDANTIC-003** MUST: Numeric constraint fields (scores, ratings, percentages) enforce bounds using Field parameters (ge, le) and reject out-of-range values during instantiation.
- **R-PYDANTIC-004** MUST: Datetime fields include Config class with json_encoders mapping datetime to isoformat method to ensure consistent serialization across all endpoints.
- **R-PYDANTIC-005** MUST: Mutable default fields (Dict, List) use Field with default_factory rather than direct assignment to avoid shared mutable state across model instances.
- **R-PYDANTIC-006** MUST: MCP service tool request and response schemas inherit from BaseModel with field-level validation constraints.
- **R-PYDANTIC-007** MUST: Agent reasoning decision models and state representations use BaseModel inheritance with documented field constraints.
- **R-PYDANTIC-008** MUST: CloudEvent data payloads and context structures use BaseModel for runtime type checking and JSON serialization.

### Verify

```bash
# Discover the project's dependency manifest and identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Locate the lock or resolution artifact to confirm the exact installed version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Discover the project's test suite location and execute validation-specific test cases
find . -path '*/test*' -name '*test*.py' -type f | grep -i 'validat\|model' | head -5

# Discover the project's static analysis configuration and run type checking
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' | head -1

# Verify all public API models have BaseModel inheritance
grep -r 'class.*BaseModel' --include='*.py' | grep -E '(router|endpoint|request|response|schema)' | wc -l

# Verify Field constraints are present on numeric fields
grep -r 'Field.*ge=\|Field.*le=' --include='*.py' | wc -l

# Verify datetime fields have json_encoders configuration
grep -r 'json_encoders.*datetime\|datetime.*isoformat' --include='*.py' | wc -l
```

**Accept when:**
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Static type checking in continuous integration pipeline verifies all public API models have complete type annotations
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses
- Code review checklist includes verification that new API models follow BaseModel patterns with Field constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing public API models without BaseModel inheritance are blocked by automated checks. Runtime validation failures generate structured error responses with field-level detail for API consumers.
</enforcement>