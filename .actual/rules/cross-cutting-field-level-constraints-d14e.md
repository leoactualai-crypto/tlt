# Adopt Pydantic BaseModel for Internal API Domain Validation: Field Level Constraints

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: Field-level constraints MUST be declared using the validation framework's field descriptor mechanism with explicit constraint parameters for numeric ranges, string patterns, and collection sizes.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' | head -1
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'requirements.lock' | head -1

# 2. Locate the project's test suite discovery mechanism
# Execute validation-related tests
python -m pytest tests/ -k validation -v

# 3. Discover the project's static type checking configuration
# Execute the type checker against all modules containing API model definitions
python -m mypy --config-file pyproject.toml src/

# 4. Search for all classes inheriting from the validation base model class
# Verify each includes field constraint declarations and type annotations
grep -r "class.*BaseModel" --include="*.py" src/ | grep -v "^Binary"
grep -r "Field(" --include="*.py" src/ | grep -E "(ge=|le=|min_length=|max_length=|pattern=)"
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- Code review checklist confirms validation model definitions for all new API endpoints

<enforcement>
Claude Code MUST NOT skip or defer verification. All internal API models must be validated before merge.
</enforcement>