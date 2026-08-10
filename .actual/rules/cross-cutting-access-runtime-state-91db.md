# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Access Runtime State

These rules are ALWAYS ACTIVE for all domain models representing business decisions, validations, ratings, or rule evaluations; public API contracts exposed via service endpoints with response_model declarations; integration test code that accesses agent state, task lifecycles, event contexts, or MCP tool arguments; and health check endpoints that evaluate service status based on runtime metrics.

### Rules

- **R-STATE-001** MUST: Access runtime state in integration tests using dictionary get operations with explicit fallback defaults to prevent KeyError exceptions.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, resolve the validation framework version,
#    then locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
find . -name 'pyproject.toml' -o -name 'requirements*.txt' -o -name 'Pipfile' | head -1
grep -E '(pydantic|mypy|pyright)' <manifest>
# Execute type checker: mypy . or pyright . (discovered from project config)

# 2. Discover the project's test runner configuration, then execute integration tests with coverage reporting
#    to verify state.get calls include fallback defaults and do not raise KeyError
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' | head -1
pytest --cov --tb=short -v tests/integration/
grep -r 'state\.get(' tests/ | grep -v 'state\.get([^,]*,[^)]*)' || echo 'All state.get calls have fallback defaults'

# 3. Discover the project's API documentation generation tool, then execute it to verify all service endpoints
#    with response_model parameters generate valid schema documentation
grep -r 'response_model=' . --include='*.py' | head -5
# Execute docs generation: typically 'python -m mkdocs build' or FastAPI auto-docs at /docs
```

**Accept when:**
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification steps MUST be executed before accepting code that modifies business rule models, integration tests accessing state, or service endpoint contracts.
</enforcement>