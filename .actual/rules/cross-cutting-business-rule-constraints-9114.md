# Encode Business Rules as Pydantic Models for API Contract Validation: Business Rule Constraints

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models, and agent reasoning decision structures.

### Rules

- **R-BRULE-001** MUST: Business rule constraints including numeric bounds, string patterns, required fields, default values, and enumerated types MUST be encoded as field-level metadata within schema definitions rather than imperative validation logic.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate test files that verify schema validation behavior
find . -path '*/test*' -name '*schema*' -o -path '*/test*' -name '*validation*' | head -5

# Search for API endpoint definitions and verify request/response models are annotated
grep -r 'def.*request\|def.*response\|@app\|@router' --include='*.py' | grep -v test | head -10

# Verify no untyped API endpoints exist
grep -r 'dict\[str, Any\]\|Dict\[str, Any\]' --include='*.py' | grep -E '@app|@router|def.*request' | wc -l
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Field-level constraint metadata (bounds, patterns, enumerations, required flags) is present in all schema model definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API contracts and domain models crossing service boundaries MUST include schema definitions with field-level constraints before code review approval.
</enforcement>