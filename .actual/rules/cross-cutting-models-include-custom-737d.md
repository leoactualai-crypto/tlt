# Adopt Pydantic BaseModel for Internal API Domain Validation: Models Include Custom

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations.
- **R-PYDANTIC-002** MUST: Define all API models in dedicated schema modules separate from business logic to enable reuse across service layers.
- **R-PYDANTIC-003** MUST: Use field descriptions consistently to document expected values, constraints, and business rules.
- **R-PYDANTIC-004** MAY: Models MAY include custom validators for complex cross-field validation logic that cannot be expressed through declarative constraints.
- **R-PYDANTIC-005** SHOULD: When composing models with nested structures, prefer typed model references over generic dictionaries to maintain validation guarantees throughout the object graph.
- **R-PYDANTIC-006** SHOULD: For agent decision models with conditional field requirements, document which fields are required for each decision type in the model docstring.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# 2. Inspect the lock artifact to determine exact resolved version
# (Output will vary by build tool; consult lock file directly)

# 3. Discover the project's test suite and execute validation-related tests
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | xargs grep -l 'testpaths\|tool.pytest' | head -1
python -m pytest -k 'validation or model' -v

# 4. Discover static type checking configuration
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' | head -1

# 5. Execute type checker against all modules containing API model definitions
find . -path '*/schema*' -name '*.py' -o -path '*/model*' -name '*.py' | xargs python -m mypy --strict

# 6. Search for all classes inheriting from validation base model
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__'

# 7. Verify each includes field constraint declarations and type annotations
grep -r 'Field(' --include='*.py' | grep -v '__pycache__' | wc -l
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- All API models are defined in dedicated schema modules separate from business logic
- Field descriptions are present and document expected values, constraints, and business rules
- Nested model compositions use typed model references rather than generic dictionaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist requires validation model definitions for all new API endpoints. Static type checking in continuous integration pipeline verifies model type annotations. Integration tests validate that API endpoints enforce model constraints and return structured validation errors. Pull requests introducing API endpoints without validated models are blocked until models are added.
</enforcement>