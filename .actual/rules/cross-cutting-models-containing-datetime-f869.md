# Enforce Pydantic BaseModel for Public API Contract Validation: Models Containing Datetime

These rules are ALWAYS ACTIVE for all public API endpoint models, MCP service tool schemas, agent reasoning decision models, CloudEvent data payloads, and domain entities exposed through public service interfaces.

### Rules

- **R-PYDANTIC-001** MUST: Models containing datetime fields MUST define json_encoders in Config class to serialize datetime objects to ISO format strings.

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
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with datetime fields include Config class with json_encoders mapping datetime to isoformat method
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code
- Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses
- Static type checking verifies all public API models have complete type annotations

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API models containing datetime fields MUST be audited for Config.json_encoders presence before code is committed.
</enforcement>