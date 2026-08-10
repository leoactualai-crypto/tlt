# Encode Business Rules as Pydantic Models for API Contract Validation: Field Definitions Include

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models, and agent reasoning decision structures.

### Rules

- **R-FIELD-001** SHOULD: Field definitions SHOULD include human-readable descriptions that serve as inline documentation for API consumers and code generation tools.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect lock file to determine exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate test files that verify schema validation behavior
find . -path '*/test*' -name '*schema*' -o -path '*/test*' -name '*validation*' | head -5

# Search for API endpoint definitions and verify request/response models have schema types
grep -r "@app\.\(get\|post\|put\|delete\)" --include="*.py" | head -10

# Verify field definitions include descriptions
grep -r "Field(.*description=" --include="*.py" | wc -l
```

**Accept when:**
- All public API endpoints define request and response models using Pydantic schema definitions with explicit type annotations and field constraints
- Field definitions include `description` parameters that document the purpose and constraints of each field
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Human-readable descriptions are present on fields crossing service boundaries and exposed in API documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API contracts MUST include field descriptions before code review approval.
</enforcement>