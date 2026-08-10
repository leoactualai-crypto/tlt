# Adopt Pydantic BaseModel for Internal API Domain Validation: Optional Fields Declared

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: Optional fields MUST be declared with explicit optional type annotations and default values or None.

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
mypy --config-file pyproject.toml $(find . -path ./venv -prune -o -name '*.py' -type f | grep -E '(schema|model|domain)' | head -20)

# 5. Search for all classes inheriting from BaseModel
grep -r 'class.*BaseModel' --include='*.py' | grep -v test | grep -v __pycache__

# 6. Verify each BaseModel includes field constraint declarations
grep -r 'Field(' --include='*.py' | grep -v test | grep -v __pycache__
```

**Accept when:**
- All internal API request and response models inherit from Pydantic BaseModel with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- Optional fields are declared with explicit Optional type hints and default values or None

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API models must pass type checking and validation tests before acceptance.
</enforcement>