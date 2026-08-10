# Encode Business Rules as Pydantic Models for API Contract Validation: Schema Models Provide

These rules are ALWAYS ACTIVE for all HTTP API request and response models, inter-service message contracts, domain entity models crossing service boundaries, configuration models, and agent reasoning decision structures.

### Rules

- **R-SCHEMA-001** MUST: Schema models MUST provide serialization capabilities for datetime objects, enumerations, and custom types to ensure consistent wire format across service boundaries.
- **R-SCHEMA-002** MUST: All public API endpoints MUST define request and response models using schema definitions with explicit type annotations and field constraints.
- **R-SCHEMA-003** MUST: Schema models MUST be defined in dedicated modules separate from business logic implementation, typically colocated with the service or domain boundary they represent.
- **R-SCHEMA-004** SHOULD: Use field-level constraint metadata to encode numeric bounds, string length limits, and enumeration values rather than implementing these checks in business logic.
- **R-SCHEMA-005** SHOULD: For datetime fields, configure serialization encoders to produce consistent wire formats across all services.
- **R-SCHEMA-006** SHOULD: When schema models grow complex, decompose them into smaller composable models that can be reused across multiple API contracts.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library in use
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name '*.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Locate and execute the project's test suite to verify schema validation behavior
find . -path '*/test*' -name '*schema*' -type f | head -5

# Search the codebase for API endpoint definitions and verify schema type annotations
grep -r '@app\|@router\|@api' --include='*.py' | grep -v '.pyc' | head -10

# Use static analysis to detect untyped endpoints
grep -r 'def.*request.*:' --include='*.py' | grep -v 'Dict\|BaseModel\|Pydantic' | head -10
```

**Accept when:**
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries
- Datetime fields are configured with serialization encoders producing consistent wire formats
- Schema models are located in dedicated modules separate from business logic

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis failures block pull request merging until schema definitions are added. Code review identifies missing or incomplete schema definitions and requests changes before approval.
</enforcement>