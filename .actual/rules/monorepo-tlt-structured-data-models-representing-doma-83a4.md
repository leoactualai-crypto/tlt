# Pydantic BaseModel for Structured Data Validation and Serialization: Structured Data Models Representing Domain Entities

These rules are ALWAYS ACTIVE for all structured data models representing domain entities, API contracts, configuration schemas, data transfer objects, and state models that cross module or service boundaries.

### Rules

- **R-PYDANTIC-001** MUST: All structured data models representing domain entities, API contracts, configuration schemas, or data transfer objects MUST inherit from Pydantic BaseModel to enforce runtime type validation and enable automatic serialization.
- **R-PYDANTIC-002** MUST: All model fields MUST have explicit type annotations to enable both static type checking and runtime validation.
- **R-PYDANTIC-003** MUST: For optional fields with complex defaults (dicts, lists), use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues.
- **R-PYDANTIC-004** SHOULD: When defining models with datetime fields that need JSON serialization, use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat().
- **R-PYDANTIC-005** SHOULD: Import BaseModel and Field from the validation library's root namespace; verify the exact import paths and available Field parameters in the locked version's documentation before use.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact; identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Confirm BaseModel and Field are available in the locked version
grep -r "from pydantic import BaseModel" . --include="*.py" | head -3

# 3. Discover the project's test suite location and execute model validation tests
find . -path '*/test*' -name '*test*.py' -type f | grep -i model | head -5

# 4. Discover the project's static analysis configuration and execute type checking
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l mypy 2>/dev/null | head -1

# 5. Verify all structured data models inherit from BaseModel
grep -r "class.*BaseModel" . --include="*.py" | wc -l

# 6. Verify model fields have explicit type annotations
grep -r "class.*BaseModel" . --include="*.py" -A 10 | grep -E "^\s+\w+:\s+" | head -10
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from Pydantic BaseModel
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- All model fields have explicit type annotations verified by static type checking
- Complex default fields use Field(default_factory=...) to avoid mutable default issues

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if structured data models lack BaseModel inheritance or type annotations. CI pipeline MUST fail if static type checking detects missing annotations on model fields.
</enforcement>