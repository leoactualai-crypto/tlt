# Pydantic BaseModel for Structured Data Validation and Serialization: Models Requiring Custom Json Serialization Datetime

These rules are ALWAYS ACTIVE for all structured data models in domain, API, configuration, and data transfer object modules that cross service boundaries or require runtime validation.

### Rules

- **R-PYDANTIC-001** SHOULD: Models requiring custom JSON serialization (datetime, enums, custom types) SHOULD define a nested Config class with json_encoders mapping types to serialization functions.
- **R-PYDANTIC-002** SHOULD: When defining models with datetime fields that need JSON serialization, use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat().
- **R-PYDANTIC-003** SHOULD: For optional fields with complex defaults (dicts, lists), always use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues.
- **R-PYDANTIC-004** MUST: Import BaseModel and Field from the validation library's root namespace; verify the exact import paths and available Field parameters in the locked version's documentation before use.
- **R-PYDANTIC-005** MUST: All structured data models in domain, API, and configuration modules inherit from the validation library's BaseModel class.
- **R-PYDANTIC-006** MUST: All model fields have explicit type annotations.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact; identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Confirm BaseModel and Field are available in the locked version's documentation
# (Manual step: inspect lock artifact for exact version, then verify against official docs)

# Discover the project's test suite location and execute model validation tests
find . -path '*/test*' -name '*test*.py' -type f | grep -i model | head -5

# Discover the project's static analysis configuration and execute type checking
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# Verify all model files inherit from BaseModel
grep -r 'class.*BaseModel' --include='*.py' | grep -E '(domain|models|schema|dto)' | wc -l

# Verify all model fields have explicit type annotations
grep -r 'class.*BaseModel' -A 20 --include='*.py' | grep -E '^\s+[a-z_]+\s*:' | wc -l
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from the validation library's BaseModel class
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- All model fields have explicit type annotations verified by static type checking
- Config.json_encoders is defined for models with datetime, enum, or custom type fields
- Optional fields with complex defaults use Field(default_factory=...) pattern

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if structured data models lack BaseModel inheritance or type annotations. CI pipeline MUST fail if static type checking detects missing annotations on model fields. Runtime validation errors in production MUST trigger alerts for investigation of data integrity issues.
</enforcement>