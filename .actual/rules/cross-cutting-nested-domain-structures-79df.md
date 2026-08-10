# Adopt Pydantic BaseModel for Internal API Domain Validation: Nested Domain Structures

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-NESTED-001** SHOULD: Nested domain structures SHOULD be composed using typed references to other validated model classes rather than generic dictionaries.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -5

# 2. Locate the project's test suite discovery mechanism and execute validation-related tests
python -m pytest -k validation --collect-only
python -m pytest -k validation -v

# 3. Discover the project's static type checking configuration
find . -name 'pyproject.toml' -o -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyrightconfig.json' | head -3

# 4. Execute the type checker against all modules containing API model definitions
mypy src/ --strict 2>&1 | grep -E '(error|model|schema)' || echo 'Type checking passed'

# 5. Search for all classes inheriting from the validation base model class
grep -r 'class.*BaseModel' --include='*.py' src/ | head -20

# 6. Verify each includes field constraint declarations and type annotations
grep -A 5 'class.*BaseModel' src/ | grep -E '(Field|:.*=|ge=|le=|min_length|max_length)' | head -20
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- No generic dictionaries are used in place of typed model references in nested domain structures

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI/CD pipeline enforcement.
</enforcement>