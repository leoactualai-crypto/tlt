# Pydantic BaseModel for Structured Data Validation and Serialization: Before Implementing Any Code That Instantiates

These rules are ALWAYS ACTIVE for all code that instantiates, serializes, or validates structured data models crossing service boundaries, including domain models, API contracts, configuration schemas, and data transfer objects.

### Rules

- **R-PYDANTIC-001** MUST: Before implementing any code that instantiates or serializes Pydantic models, discover the project's dependency lock artifact, resolve the exact installed Pydantic version, and verify all Field configurations, validators, and Config options against that version's official documentation.
- **R-PYDANTIC-002** MUST: All structured data models in domain, API, and configuration modules inherit from Pydantic's BaseModel class.
- **R-PYDANTIC-003** MUST: All model fields have explicit type annotations.
- **R-PYDANTIC-004** MUST: Import BaseModel and Field from the validation library's root namespace; verify the exact import paths and available Field parameters in the locked version's documentation before use.
- **R-PYDANTIC-005** SHOULD: When defining models with datetime fields that need JSON serialization, use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat().
- **R-PYDANTIC-006** SHOULD: For optional fields with complex defaults (dicts, lists), always use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues.
- **R-PYDANTIC-007** MAY: Performance-critical hot paths may request exception via architecture review with profiling data demonstrating validation overhead impact.
- **R-PYDANTIC-008** MAY: Legacy integration points may use alternative serialization temporarily with documented migration plan.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Identify the validation library version and confirm BaseModel and Field are available
grep -r "pydantic" pyproject.toml poetry.lock requirements.txt setup.py 2>/dev/null | head -20

# 2. Discover the project's test suite location and execute model validation tests
find . -type f -name "*test*model*.py" -o -name "*test*validation*.py" | head -10

# 3. Discover the project's static analysis configuration and execute type checking
find . -type f \( -name "pyproject.toml" -o -name "mypy.ini" -o -name ".mypy.ini" -o -name "setup.cfg" \) | xargs grep -l "mypy\|type" 2>/dev/null

# 4. Verify all structured data models inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E "(domain|model|schema|dto)" | wc -l

# 5. Verify model instantiation with invalid data raises validation errors
grep -r "ValidationError" --include="*.py" | head -10

# 6. Verify JSON serialization of models with datetime fields produces ISO 8601 formatted strings
grep -r "json_encoders\|isoformat" --include="*.py" | head -10
```

**Accept when:**
- The exact Pydantic version is identified from the project's lock artifact and all Field configurations are verified against that version's official documentation
- All structured data models in domain, API, and configuration modules inherit from BaseModel
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- All model fields have explicit type annotations verified by static type checking
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic
- Integration tests verify model validation behavior and JSON serialization correctness

<enforcement>
Claude Code MUST NOT skip or defer verification of the exact Pydantic version and official documentation before implementing any code that instantiates or serializes Pydantic models. Code review MUST block merge if structured data models lack BaseModel inheritance or type annotations. CI pipeline MUST fail if static type checking detects missing annotations on model fields.
</enforcement>