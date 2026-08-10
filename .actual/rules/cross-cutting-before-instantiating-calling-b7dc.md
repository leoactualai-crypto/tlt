# Adopt Pydantic BaseModel for Internal API Domain Validation: Before Instantiating Calling

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint request and response models, agent reasoning decision structures, MCP service schemas, CloudEvent models, and health check response structures.

### Rules

- **R-PYDANTIC-001** MUST: Before instantiating or calling any validation framework class or API, discover the project's dependency lock artifact, resolve the exact installed version, and verify all used APIs exist in that version's official documentation.
- **R-PYDANTIC-002** MUST: Define all API models in dedicated schema modules separate from business logic to enable reuse across service layers and maintain clear separation of concerns.
- **R-PYDANTIC-003** MUST: Use field descriptions consistently to document expected values, constraints, and business rules in all model definitions.
- **R-PYDANTIC-004** SHOULD: For agent decision models with conditional field requirements, document which fields are required for each decision type in the model docstring to guide consumers.
- **R-PYDANTIC-005** SHOULD: When composing models with nested structures, prefer typed model references over generic dictionaries to maintain validation guarantees throughout the object graph.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
# Locate the project's test suite discovery mechanism and execute validation-related tests

# 2. Discover the project's static type checking configuration
# Execute the type checker against all modules containing API model definitions

# 3. Discover the project's code search or analysis tooling
# Search for all classes inheriting from the validation base model class
# Verify each includes field constraint declarations and type annotations
```

**Accept when:**
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs

<enforcement>
Clause R-PYDANTIC-001 is mandatory before any code instantiation. Claude Code MUST NOT skip version discovery and verification steps. All other rules MUST be verified during code review and static analysis phases.
</enforcement>