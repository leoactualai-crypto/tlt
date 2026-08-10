# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Encode Business Rules

These rules are ALWAYS ACTIVE for all business logic modules that encode domain rules, validation constraints, or decision workflows.

### Rules

- **R-ENCODE-001** MUST: Encode all business rules as Pydantic BaseModel subclasses with Field constraints specifying description, ge, le, and default values.
- **R-ENCODE-002** MUST: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-ENCODE-003** MUST: Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- **R-ENCODE-004** MUST: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-ENCODE-005** SHOULD: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-ENCODE-006** SHOULD: Access runtime state in integration tests via dictionary get operations with fallback defaults to prevent KeyError exceptions when state structure evolves.
- **R-ENCODE-007** SHOULD: Expose business rule models as response_model parameters in service endpoints to enable automatic OpenAPI documentation and contract-first API design.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
type_check_script=$(find . -name "*.sh" -o -name "Makefile" | xargs grep -l "type.*check\|mypy\|pyright" 2>/dev/null | head -1)
if [ -n "$type_check_script" ]; then bash "$type_check_script"; fi

# 3. Discover the project's test runner configuration
# 4. Execute integration tests with coverage reporting to verify state.get calls include fallback defaults
test_config=$(find . -name "pytest.ini" -o -name "pyproject.toml" -o -name "setup.cfg" | head -1)
if [ -n "$test_config" ]; then pytest --cov --tb=short -v; fi

# 5. Discover the project's API documentation generation tool
# 6. Execute it to verify all service endpoints with response_model parameters generate valid schema documentation
api_gen=$(find . -name "*.py" | xargs grep -l "FastAPI\|response_model" 2>/dev/null | head -1)
if [ -n "$api_gen" ]; then python -m pytest --collect-only -q | grep -i "openapi\|schema"; fi
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- All domain models representing business decisions, validations, ratings, or rule evaluations follow the Pydantic BaseModel pattern with documented constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All business rule models MUST be validated against these rules before merge. Integration tests MUST pass with state.get fallback patterns. Type checking MUST complete successfully.
</enforcement>