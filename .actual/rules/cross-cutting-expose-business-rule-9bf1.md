# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Expose Business Rule

These rules are ALWAYS ACTIVE for all business logic modules that encode domain rules, validation constraints, or decision workflows, including all domain models representing business decisions, validations, ratings, or rule evaluations; public API contracts exposed via service endpoints with response_model declarations; integration test code that accesses agent state, task lifecycles, event contexts, or MCP tool arguments; and health check endpoints that evaluate service status based on runtime metrics.

### Rules

- **R-EXPOSE-001** MUST: Expose business rule models as public API contracts in response_model parameters for service endpoints.
- **R-EXPOSE-002** MUST: Inherit all business rule models from BaseModel with Field constraints specifying validation rules.
- **R-EXPOSE-003** MUST: Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- **R-EXPOSE-004** MUST: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-EXPOSE-005** MUST: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-EXPOSE-006** MUST: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-EXPOSE-007** MUST: Access runtime state in integration tests via dictionary get operations with explicit fallback defaults to prevent KeyError exceptions.
- **R-EXPOSE-008** SHOULD: Limit nesting depth to 3-4 levels in complex nested models to prevent performance degradation or stack overflow.
- **R-EXPOSE-009** SHOULD: Use lazy validation for optional nested models to improve performance.
- **R-EXPOSE-010** SHOULD: Add explicit assertions for critical state keys before fallback logic in integration tests.
- **R-EXPOSE-011** SHOULD: Log warnings when fallback defaults are used in integration tests to detect missing state keys.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1

# 3. Discover the project's test runner configuration and execute integration tests with coverage reporting
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1

# 4. Verify state.get calls include fallback defaults and do not raise KeyError
grep -r 'state\.get' --include='*.py' | grep -v 'state\.get.*,' && echo 'FAIL: Found state.get without fallback defaults' || echo 'PASS: All state.get calls have fallback defaults'

# 5. Discover the project's API documentation generation tool and execute it
find . -name 'openapi.json' -o -name 'swagger.json' -o -name 'docs' -type d | head -1

# 6. Verify all service endpoints expose business rule models as response_model parameters
grep -r 'response_model' --include='*.py' | wc -l
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- All business rule models include Field descriptions and validation constraints
- No circular import issues exist between domain models
- Nesting depth in complex models does not exceed 4 levels

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before accepting code changes. Type checking, integration tests, and API documentation generation must pass in the continuous integration pipeline.
</enforcement>