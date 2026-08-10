# Encode Business Rules as Pydantic Models for API Contract Validation: Public Contracts Exposed

These rules are ALWAYS ACTIVE for all public API contracts exposed through HTTP endpoints or inter-service communication channels, domain entity models that cross service boundaries, configuration models loaded from external sources, and agent reasoning decision structures.

### Rules

- **R-PYDANTIC-001** MUST: All public API contracts exposed through HTTP endpoints or inter-service communication channels MUST be defined as declarative schema models with explicit type annotations for every field.
- **R-PYDANTIC-002** MUST: Define schema models in dedicated modules separate from business logic implementation, typically colocated with the service or domain boundary they represent.
- **R-PYDANTIC-003** MUST: Use field-level constraint metadata to encode numeric bounds, string length limits, and enumeration values rather than implementing these checks in business logic.
- **R-PYDANTIC-004** SHOULD: For datetime fields, configure serialization encoders to produce consistent wire formats across all services.
- **R-PYDANTIC-005** SHOULD: When schema models grow complex, decompose them into smaller composable models that can be reused across multiple API contracts.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect lock file to determine exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'requirements.lock' | head -1

# Locate test files that verify schema validation behavior
find . -path '*/test*' -name '*test*.py' -type f | grep -i 'schema\|model\|validat' | head -5

# Search for API endpoint definitions and verify request/response models are annotated
grep -r 'def.*request\|def.*response\|@app\|@router' --include='*.py' | grep -v test | head -10

# Verify no untyped endpoints exist
grep -r 'dict\[str, Any\]\|Dict\[str, Any\]' --include='*.py' | grep -E 'request|response|endpoint' | wc -l
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Field-level constraints (numeric bounds, string length limits, enumerations) are encoded in schema model definitions, not in business logic
- Datetime fields use consistent serialization encoders across all services

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API contracts MUST be validated against these rules before code is committed. Static analysis failures block pull request merging until schema definitions are added.
</enforcement>