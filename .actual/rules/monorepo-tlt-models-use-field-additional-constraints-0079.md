# Pydantic BaseModel for Structured Data Validation and Serialization: Models Use Field Additional Constraints Min

These rules are ALWAYS ACTIVE for all structured data models, API contracts, configuration schemas, and data transfer objects that cross service boundaries or require runtime validation guarantees.

### Rules

- **R-PYDANTIC-001** MAY: Models MAY use Field() with additional constraints (min_length, max_length, ge, le) for fine-grained validation beyond type checking.
- **R-PYDANTIC-002** MUST: All domain models representing business entities (Guild, Event, User, RSVP) inherit from BaseModel.
- **R-PYDANTIC-003** MUST: All API request and response models for service endpoints inherit from BaseModel with explicit type annotations.
- **R-PYDANTIC-004** MUST: All configuration schemas for service settings and feature flags inherit from BaseModel.
- **R-PYDANTIC-005** MUST: All data transfer objects passed between agent nodes or service layers inherit from BaseModel.
- **R-PYDANTIC-006** MUST: All state models for agent workflows requiring validation inherit from BaseModel.
- **R-PYDANTIC-007** SHOULD: When defining models with datetime fields that need JSON serialization, use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat().
- **R-PYDANTIC-008** MUST: For optional fields with complex defaults (dicts, lists), always use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues.
- **R-PYDANTIC-009** MUST: Import BaseModel and Field from the validation library's root namespace; verify the exact import paths and available Field parameters in the locked version's documentation before use.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact; identify the validation library version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Confirm BaseModel and Field are available in the locked version
grep -r "from pydantic import BaseModel, Field" . --include="*.py" | head -10

# 3. Discover the project's test suite location and execute model validation tests
find . -path '*/test*' -name '*test*.py' -o -path '*/tests/*' -name '*.py' | grep -i model | head -10

# 4. Verify all structured data models inherit from BaseModel
grep -r "class.*BaseModel" . --include="*.py" | grep -E '(domain|api|config|model)' | head -20

# 5. Discover the project's static analysis configuration and execute type checking
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' -o -name 'setup.cfg' | head -5

# 6. Verify model fields have explicit type annotations
grep -r "class.*BaseModel" . --include="*.py" -A 10 | grep -E ':\s*(str|int|bool|datetime|List|Dict|Optional)' | head -20
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from BaseModel
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- All model fields have explicit type annotations verified by static type checking
- Field() constraints (min_length, max_length, ge, le) are applied where fine-grained validation is required
- Complex default values use Field(default_factory=...) to avoid mutable default issues
- BaseModel and Field imports are verified against the locked version's documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval. Violations block merge unless an explicit exception is granted via architecture review with documented rationale and migration timeline.
</enforcement>