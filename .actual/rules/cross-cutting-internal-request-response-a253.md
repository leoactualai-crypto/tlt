# Adopt Pydantic BaseModel for Internal API Domain Validation: Internal Request Response

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check responses.

### Rules

- **R-PYDANTIC-001** MUST: All internal API request and response models MUST be defined as classes inheriting from the validation framework's base model class.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# 2. Inspect the lock artifact to determine exact resolved version
cat poetry.lock | grep -A 5 'name = "pydantic"' || cat requirements.txt | grep pydantic

# 3. Locate and execute the project's test suite for validation-related tests
pytest tests/ -k validation -v

# 4. Discover static type checking configuration and execute type checker
mypy --config-file pyproject.toml $(find . -path ./venv -prune -o -name '*.py' -type f | grep -E '(schema|model|api)' | head -20)

# 5. Search for all classes inheriting from validation base model
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | grep -E '(schema|model|api)'

# 6. Verify each includes field constraint declarations and type annotations
grep -r 'Field(' --include='*.py' | grep -v '__pycache__' | wc -l
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- Code review checklist confirms validation model definitions for all new API endpoints
- Integration tests validate that API endpoints enforce model constraints and return structured validation errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All internal API models must be validated against these rules before acceptance.
</enforcement>