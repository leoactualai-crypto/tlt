# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Implement Health Check

These rules are ALWAYS ACTIVE for all business logic modules that encode domain rules, validation constraints, decision workflows, public API contracts, integration tests accessing agent state, and health check endpoints evaluating service status based on runtime metrics.

### Rules

- **R-HEALTH-001** SHOULD: Implement health check endpoints that return structured dictionaries with conditional status degradation based on queue size, agent availability, and task metrics.
- **R-HEALTH-002** MUST: Encode all domain models representing business decisions, validations, ratings, or rule evaluations as Pydantic BaseModel subclasses with Field constraints specifying validation rules.
- **R-HEALTH-003** MUST: Expose business rule models as response_model parameters in service endpoints to enable automatic OpenAPI documentation and contract-first API design.
- **R-HEALTH-004** MUST: Access runtime state in integration tests using dictionary get operations with explicit fallback defaults to prevent KeyError exceptions when state structure evolves.
- **R-HEALTH-005** SHOULD: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-HEALTH-006** SHOULD: Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- **R-HEALTH-007** SHOULD: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-HEALTH-008** MAY: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-HEALTH-009** MUST: Maintain backward compatibility with untyped dictionary responses during migration period (EXC-001).

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
type_check_script=$(find . -name "*.sh" -o -name "Makefile" -o -name "pyproject.toml" | xargs grep -l "mypy\|pyright" | head -1)
# Execute type checking to verify BaseModel completeness

# 3. Discover the project's test runner configuration
# 4. Execute integration tests with coverage reporting to verify state.get calls include fallback defaults
test_runner=$(find . -name "pytest.ini" -o -name "setup.cfg" -o -name "pyproject.toml" | head -1)
# Run integration tests with coverage to verify no KeyError exceptions

# 5. Discover the project's API documentation generation tool
# 6. Execute it to verify all service endpoints with response_model parameters generate valid schema documentation
api_doc_tool=$(find . -name "*.py" | xargs grep -l "FastAPI\|response_model" | head -1)
# Generate API documentation and verify schema completeness
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- Health check endpoints return structured dictionaries with conditional degradation logic based on queue size, agent availability, and task completion metrics
- All domain models include Field descriptions and validation constraints in their definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking, integration test execution, and API documentation generation MUST complete successfully before accepting changes to business rule models or health check endpoints.
</enforcement>