# Pydantic BaseModel for Structured Data Validation and Serialization: Fields Complex Defaults Factory Functions Use

These rules are ALWAYS ACTIVE for all structured data models, domain entities, API contracts, configuration schemas, and data transfer objects that cross service boundaries or require runtime validation.

### Rules

- **R-PYDANTIC-001** SHOULD: Fields with complex defaults or factory functions SHOULD use `Field(default_factory=...)` to ensure proper instantiation per model instance and avoid shared mutable default issues.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Identify the validation library version and confirm BaseModel and Field are available
grep -r "pydantic" pyproject.toml requirements.txt setup.py 2>/dev/null | head -5

# 2. Verify all structured data models inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E "(domain|models|schemas|api)" | head -10

# 3. Verify Field(default_factory=...) usage for complex defaults
grep -r "Field(default_factory" --include="*.py" | head -10

# 4. Identify any mutable defaults (list, dict) not using default_factory
grep -r ":\s*\(list\|dict\)\s*=\s*\[\|{" --include="*.py" | grep -v "default_factory" | head -5

# 5. Run static type checking to verify model field annotations
python -m mypy . --strict 2>&1 | grep -E "(BaseModel|Field)" | head -10

# 6. Locate and execute model validation tests
find . -name "*test*model*.py" -o -name "*test*validation*.py" | head -5
```

**Accept when:**
- All structured data models in domain, API, and configuration modules inherit from `BaseModel`
- Complex default fields (lists, dicts, datetime objects) use `Field(default_factory=...)` syntax
- Model instantiation with invalid data raises validation errors at runtime
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings
- Static type checking confirms all model fields have explicit type annotations
- No mutable defaults (bare `[]` or `{}`) appear on model fields outside of `default_factory`

<enforcement>
Clause MUST NOT skip or defer verification. Code review MUST confirm Field(default_factory=...) usage before merging changes to structured data models. CI pipeline MUST execute type checking and validation tests.
</enforcement>