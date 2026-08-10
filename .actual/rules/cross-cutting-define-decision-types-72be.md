# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Define Decision Types

These rules are ALWAYS ACTIVE for all domain models representing business decisions, validations, ratings, or rule evaluations; public API contracts exposed via service endpoints with response_model declarations; integration test code that accesses agent state, task lifecycles, event contexts, or MCP tool arguments; and health check endpoints that evaluate service status based on runtime metrics.

### Rules

- **R-PYDANTIC-001** MUST: Define decision types, ratings, and domain enumerations as typed fields with validation constraints rather than untyped string or integer values.
- **R-PYDANTIC-002** MUST: Use Pydantic BaseModel subclasses with Field constraints for all domain models representing business decisions, validations, ratings, or rule evaluations.
- **R-PYDANTIC-003** MUST: Expose business rule models as response_model parameters in service endpoints to enable automatic OpenAPI documentation and contract-first API design.
- **R-PYDANTIC-004** MUST: Access runtime state in integration tests via dictionary get operations with explicit fallback defaults to prevent KeyError exceptions.
- **R-PYDANTIC-005** SHOULD: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-PYDANTIC-006** SHOULD: Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- **R-PYDANTIC-007** SHOULD: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-PYDANTIC-008** SHOULD: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-PYDANTIC-009** MAY: Request exception approval from engineering lead with documented justification and migration timeline for legacy endpoints that must maintain backward compatibility with untyped dictionary responses during migration period (EXC-001).

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
type_check_script=$(find . -name "*.sh" -o -name "Makefile" -o -name "pyproject.toml" | xargs grep -l "mypy\|pyright\|type" | head -1)
# Execute type checking: $type_check_script

# 3. Discover the project's test runner configuration
# 4. Execute integration tests with coverage reporting to verify state.get calls include fallback defaults
test_runner=$(find . -name "pytest.ini" -o -name "setup.cfg" -o -name "pyproject.toml" | head -1)
# Execute tests with coverage: pytest --cov --tb=short

# 5. Discover the project's API documentation generation tool
# 6. Execute it to verify all service endpoints with response_model parameters generate valid schema documentation
api_doc_tool=$(find . -name "*.py" | xargs grep -l "FastAPI\|response_model" | head -1)
# Generate and validate API schema documentation
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- Health check endpoints return consistent structured responses with conditional degradation logic based on runtime metrics

<enforcement>
Claude Code MUST NOT skip or defer verification. All business rule models MUST be validated against this rule set before merge. Integration tests MUST pass with state.get fallback patterns verified. Type checking MUST pass without errors.
</enforcement>