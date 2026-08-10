# Adopt Pydantic BaseModel for Internal API Domain Validation: Domain Models Include

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-DOMAIN-001** SHOULD: Domain models SHOULD include field descriptions using the validation framework's description parameter to document expected values and constraints.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Locate the project's test suite discovery mechanism and execute validation-related tests
python -m pytest tests/ -k validation -v

# Discover the project's static type checking configuration
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' | head -1

# Execute the type checker against all modules containing API model definitions
mypy src/ --strict

# Search for all classes inheriting from the validation base model class
grep -r "class.*BaseModel" src/ --include="*.py"

# Verify each includes field constraint declarations and type annotations
grep -A 5 "class.*BaseModel" src/ | grep -E "Field|:.*=" 
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- All domain model fields include descriptions documenting expected values and business constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist requires validation model definitions for all new API endpoints. Static type checking in CI pipeline verifies model type annotations. Integration tests validate that API endpoints enforce model constraints and return structured validation errors. Pull requests introducing API endpoints without validated models are blocked until models are added.
</enforcement>