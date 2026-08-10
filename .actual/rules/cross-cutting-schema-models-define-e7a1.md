# Encode Business Rules as Pydantic Models for API Contract Validation: Schema Models Define

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models, and agent reasoning decision structures.

### Rules

- **R-SCHEMA-001** MAY: Schema models MAY define custom validators for cross-field business rules that cannot be expressed through single-field constraints.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name '*.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Locate and execute the project's test suite to verify schema validation behavior
find . -path '*/test*' -name '*schema*' -type f | head -5

# Search the codebase for API endpoint definitions and verify schema type annotations
grep -r "@app\.\(get\|post\|put\|delete\)" --include="*.py" | head -10
grep -r "def.*request.*:" --include="*.py" | grep -E "(BaseModel|pydantic)" | head -10

# Use static analysis to detect untyped endpoints
grep -r "dict\[str, Any\]" --include="*.py" | grep -E "(request|response)" | head -5
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Custom validators are documented with rationale for cross-field business rules that cannot be expressed through single-field constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API contracts MUST include schema definitions. Static analysis failures block pull request merging until schema definitions are added.
</enforcement>