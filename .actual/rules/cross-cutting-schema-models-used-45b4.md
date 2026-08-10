# Encode Business Rules as Pydantic Models for API Contract Validation: Schema Models Used

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models loaded from external sources, and agent reasoning decision structures.

### Rules

- **R-SCHEMA-001** MUST: Schema models used for API contracts MUST support automatic validation at deserialization time, rejecting invalid input before it reaches business logic layers.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name '*.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Locate test files that verify schema validation behavior
find . -path '*/test*' -name '*schema*' -o -path '*/test*' -name '*validation*' | head -5

# Search the codebase for API endpoint definitions and verify request/response models are annotated
grep -r "@app\.\(get\|post\|put\|delete\|patch\)" --include="*.py" | head -10

# Use static analysis to detect untyped endpoints
grep -r "def.*request.*:.*dict" --include="*.py" | head -5
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- The project's dependency manifest declares a schema validation library (e.g., Pydantic) and the lock file shows a resolved version

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API contracts MUST include schema model definitions with automatic validation. Violations block pull request merging until remediated.
</enforcement>