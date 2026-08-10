# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Models Used Endpoints

These rules are ALWAYS ACTIVE for all domain models representing events, decisions, API contracts, and workflow states that cross service boundaries or are used in API endpoints.

### Rules

- **R-PYDANTIC-001** MUST: Models used in API endpoints MUST be declared as response models to enable automatic serialization validation and OpenAPI schema generation.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# Locate and execute the project's type checking verification script
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# Discover the project's test runner configuration and execute integration tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# Discover the project's API documentation generation tool and verify schema generation
grep -r 'openapi\|swagger\|fastapi' . --include='*.py' --include='*.toml' | head -5
```

**Accept when:**
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation
- API endpoints declare response models and generate valid OpenAPI schemas automatically
- Static type checking in the continuous integration pipeline detects missing model annotations

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints and external event handlers crossing service boundaries MUST declare validation models. Pull requests introducing unvalidated domain models are subject to rejection in code review.
</enforcement>