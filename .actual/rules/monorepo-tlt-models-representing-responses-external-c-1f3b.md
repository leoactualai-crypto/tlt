# Pydantic BaseModel for Structured Data Validation and Serialization: Models Representing Responses External Contracts Include

These rules are ALWAYS ACTIVE for all structured data models representing API responses, external contracts, domain entities, configuration schemas, and data transfer objects crossing service boundaries.

### Rules

- **R-PYDANTIC-001** SHOULD: Models representing API responses or external contracts SHOULD include docstrings describing their purpose and usage context.
- **R-PYDANTIC-002** MUST: All structured data models in domain, API, and configuration modules MUST inherit from the validation library's BaseModel class.
- **R-PYDANTIC-003** MUST: All model fields MUST have explicit type annotations.
- **R-PYDANTIC-004** SHOULD: When defining models with datetime fields that need JSON serialization, SHOULD use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat().
- **R-PYDANTIC-005** MUST: For optional fields with complex defaults (dicts, lists), MUST always use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues.
- **R-PYDANTIC-006** MUST: Import BaseModel and Field from the validation library's root namespace; MUST verify the exact import paths and available Field parameters in the locked version's documentation before use.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact; identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# 2. Confirm BaseModel and Field are available in the locked version
grep -E '(pydantic|validation)' <lock-artifact> | head -5

# 3. Discover the project's test suite location and execute model validation tests
find . -path '*/test*' -name '*model*' -type f | head -5

# 4. Discover the project's static analysis configuration and execute type checking
find . -name 'mypy.ini' -o -name 'pyproject.toml' -o -name '.pylintrc' | head -1

# 5. Verify all structured data models inherit from BaseModel
grep -r 'class.*BaseModel' --include='*.py' | wc -l

# 6. Verify model instantiation with invalid data raises validation errors
python -m pytest <test-suite-path> -v -k 'validation or model'

# 7. Verify JSON serialization of models with datetime fields produces ISO 8601 formatted strings
python -c "from datetime import datetime, timezone; import json; print('ISO format test')"
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from the validation library's BaseModel class
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- All model fields have explicit type annotations verified by static type checking
- Models representing API responses or external contracts include docstrings describing their purpose and usage context
- Optional fields with complex defaults use Field(default_factory=...) to avoid mutable default issues

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if structured data models lack BaseModel inheritance or type annotations. CI pipeline MUST fail if static type checking detects missing annotations on model fields. Runtime validation errors in production MUST trigger alerts for investigation of data integrity issues.
</enforcement>