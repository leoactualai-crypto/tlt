# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Include Reasoning Confidence

These rules are ALWAYS ACTIVE for all domain models representing business decisions, validations, ratings, rule evaluations, public API contracts exposed via service endpoints, integration test code accessing agent state and task lifecycles, and health check endpoints evaluating service status based on runtime metrics.

### Rules

- **R-PYDANTIC-001** MUST: Encode all business rules, validations, ratings, and decision workflows as Pydantic BaseModel subclasses with Field constraints specifying validation rules (ge/le bounds, enumerations, string patterns).
- **R-PYDANTIC-002** MUST: Include Field descriptions in all business rule models to enable self-documenting code and automatic OpenAPI schema generation.
- **R-PYDANTIC-003** MUST: Expose business rule models as response_model parameters in service endpoint declarations to enable automatic API documentation and contract-first design.
- **R-PYDANTIC-004** MUST: Use dictionary get operations with explicit fallback defaults in integration tests when accessing runtime state, preventing KeyError exceptions when state structure evolves.
- **R-PYDANTIC-005** MAY: Include reasoning, confidence, and metadata fields in decision models to support explainability and audit requirements.
- **R-PYDANTIC-006** SHOULD: Place business rule models in domain-specific modules separate from service endpoints to enable reuse across multiple API routes and test files without circular dependencies.
- **R-PYDANTIC-007** SHOULD: Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- **R-PYDANTIC-008** SHOULD: Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- **R-PYDANTIC-009** SHOULD: Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.
- **R-PYDANTIC-010** MUST NOT: Encode business rules as untyped dictionaries or ad-hoc procedural validation logic in service endpoints or integration tests.
- **R-PYDANTIC-011** MUST NOT: Access runtime state in integration tests without explicit fallback defaults that prevent KeyError exceptions.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# 2. Locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
find . -name 'mypy.ini' -o -name '.mypy.ini' -o -name 'pyproject.toml' | xargs grep -l 'mypy' 2>/dev/null | head -1
# Then run: mypy --config-file <config> <source_dir>

# 3. Discover the project's test runner configuration and execute integration tests with coverage reporting
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1
# Then run: pytest --cov=. --cov-report=term-missing -v

# 4. Verify state.get calls include fallback defaults and do not raise KeyError
grep -r 'state\.get' . --include='*.py' | grep -v 'state\.get.*,' && echo 'FAIL: state.get calls missing fallback defaults' || echo 'PASS: All state.get calls include defaults'

# 5. Discover the project's API documentation generation tool and execute it
find . -name 'openapi.json' -o -name 'swagger.json' -o -name 'docs' -type d | head -1
# Then verify response_model parameters are present in endpoint definitions
grep -r 'response_model=' . --include='*.py' | wc -l
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions
- All BaseModel subclasses have complete type annotations verified by the project's type checking tool
- No untyped dictionary responses are exposed at service boundaries without corresponding Pydantic model definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All business rule models MUST be validated against these rules before merge. Integration tests MUST be executed to confirm state.get calls include fallback defaults. Type checking MUST pass without errors.
</enforcement>