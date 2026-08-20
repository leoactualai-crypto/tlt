# Pydantic BaseModel for Structured Data Validation and Serialization: Model Fields Include Explicit Type Annotations

These rules are ALWAYS ACTIVE for all structured data models crossing service boundaries, including domain models, API contracts, configuration schemas, data transfer objects, and agent workflow state models.

### Rules

- **R-PYDANTIC-001** MUST: All model fields MUST include explicit type annotations using typing module constructs (Optional, Dict, List, Any) to enable Pydantic's validation engine.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact; identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Confirm BaseModel and Field are available in the locked version's documentation
# (Execute after identifying exact version from lock artifact)

# Discover the project's test suite location and execute model validation tests
find . -path '*/test*' -name '*test*.py' -type f | grep -i model | head -5

# Discover the project's static analysis configuration and execute type checking
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' | head -1
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from the validation library's BaseModel class
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- Static type checking confirms all model fields have explicit type annotations
- Code review checklist verifies new data models include type annotations on every field

<enforcement>
Claude Code MUST NOT skip or defer verification. All model fields crossing service boundaries MUST have explicit type annotations before code review approval.
</enforcement>