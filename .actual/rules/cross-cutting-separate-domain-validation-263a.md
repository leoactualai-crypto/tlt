# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Separate Domain Validation

These rules are ALWAYS ACTIVE for all domain models representing business decisions, validations, ratings, or rule evaluations; public API contracts exposed via service endpoints with response_model declarations; integration test code that accesses agent state, task lifecycles, event contexts, or MCP tool arguments; and health check endpoints that evaluate service status based on runtime metrics.

### Rules

- **R-DOMAIN-001** SHOULD: Separate domain validation logic (Pydantic BaseModel subclasses) from integration testing concerns (state access patterns) and observability instrumentation (logger imports).
- **R-DOMAIN-002** MUST: Encode all business rules, validation constraints, and decision workflows as Pydantic BaseModel subclasses with Field constraints specifying validation rules (ge/le bounds, enumerations, string patterns).
- **R-DOMAIN-003** MUST: Expose business rule models as response_model parameters in service endpoint declarations to enable automatic OpenAPI documentation and contract-first API design.
- **R-DOMAIN-004** SHOULD: Access runtime state in integration tests using dictionary get operations with explicit fallback defaults to prevent KeyError exceptions when state structure evolves.
- **R-DOMAIN-005** SHOULD: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-DOMAIN-006** MUST: Use Field default_factory for mutable defaults (dictionaries, lists) to prevent shared state bugs across model instances.
- **R-DOMAIN-007** SHOULD: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-DOMAIN-008** SHOULD: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-DOMAIN-009** MUST: Include Field descriptions for all model attributes to enable self-documenting business rules and improve API usability.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# 3. Execute integration tests with coverage reporting to verify state.get calls include fallback defaults
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# 4. Discover the project's API documentation generation tool and execute it
grep -r 'fastapi\|openapi\|swagger' . --include='*.py' --include='*.toml' | head -5

# 5. Verify no BaseModel subclasses lack complete type annotations
grep -r 'class.*BaseModel' . --include='*.py' | wc -l

# 6. Verify integration tests use state.get with defaults
grep -r 'state\.get(' . --include='*.py' | grep -v 'state\.get.*,' | wc -l
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- All model attributes include Field descriptions documenting their purpose and validation constraints
- Mutable default values use default_factory to prevent shared state bugs
- Custom validators are implemented using the validation framework's decorator pattern for cross-field validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All business rule models MUST be validated against these rules before merge. Integration tests MUST pass without KeyError exceptions. Type checking MUST complete successfully.
</enforcement>