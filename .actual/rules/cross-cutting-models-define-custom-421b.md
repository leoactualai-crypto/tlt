# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Models Define Custom

These rules are ALWAYS ACTIVE for all domain models representing events, decisions, API contracts, workflow states, and data structures crossing service boundaries in this project.

### Rules

- **R-PYDANTIC-001** MAY: Models MAY define custom validators for cross-field constraints or complex business rules beyond basic type and range checks.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Locate and execute the project's type checking verification script
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# Discover the project's test runner configuration and execute integration tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# Discover the project's API documentation generation tool
find . -name 'openapi.json' -o -name 'openapi.yaml' -o -name '*openapi*' 2>/dev/null | head -1
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation
- API endpoints declare response models and generate valid OpenAPI schemas automatically
- Static type checking in the continuous integration pipeline detects missing model annotations
- Code review checklist requires validation models for all new API endpoints and external event handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain models crossing service boundaries MUST use Pydantic BaseModel with appropriate field constraints. Pull requests introducing unvalidated domain models are subject to rejection in code review.
</enforcement>