# Encode Business Rules as Pydantic Models for API Contract Validation: Schema Models Representing

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models, and agent reasoning decision structures.

### Rules

- **R-SCHEMA-001** SHOULD: Schema models representing domain entities SHOULD separate concerns by defining distinct models for creation requests, response payloads, and internal state representations.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect lock file to determine exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate test files that verify schema validation behavior
find . -path '*/test*' -name '*schema*' -o -path '*/test*' -name '*validation*' | head -5

# Search for API endpoint definitions and verify request/response models are annotated
grep -r 'def.*request\|def.*response\|@app\|@router' --include='*.py' | grep -E '(Request|Response|BaseModel)' | head -10

# Verify no untyped API endpoints exist
grep -r 'def.*->.*dict\|def.*->.*Dict' --include='*.py' | head -5
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Schema models are organized in dedicated modules separate from business logic implementation
- Field-level constraint metadata encodes numeric bounds, string length limits, and enumeration values

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API contracts MUST include schema definitions before merging. Static analysis failures block pull request merging until schema definitions are added.
</enforcement>