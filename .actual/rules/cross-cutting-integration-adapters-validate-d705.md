# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Integration Adapters Validate

These rules are ALWAYS ACTIVE for all HTTP API endpoints, background tasks, and integration adapter modules that initiate real-time message delivery to external platforms.

### Rules

- **R-ASYNC-001** MUST: Integration adapters MUST validate all inbound request payloads using structured schema models before initiating message delivery workflows.
- **R-ASYNC-002** MUST: All async message delivery methods MUST be wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility.
- **R-ASYNC-003** MUST: Async HTTP client sessions MUST be configured with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures.
- **R-ASYNC-004** MUST: Environment variable names and default values for runtime configuration MUST be documented in deployment configuration to ensure consistent behavior across environments.
- **R-ASYNC-005** MUST: All integration adapter message delivery operations MUST use async send methods without blocking calls within async functions.
- **R-ASYNC-006** MUST: HTTP API endpoints MUST declare structured response models and validation MUST reject malformed requests with appropriate error codes.
- **R-ASYNC-007** MUST: Background tasks MUST use async HTTP client sessions with documented lifecycle management and environment-based configuration.

### Verify

```bash
# Discover and execute the project's integration test suite that validates async message delivery workflows
# against mock external platform endpoints
find . -type f -name '*test*integration*' -o -name '*integration*test*' | head -5

# Discover and run the project's static analysis tooling to verify all message delivery operations
# use async/await patterns without blocking calls
find . -type f -name 'pyproject.toml' -o -name 'setup.py' -o -name 'setup.cfg' | xargs grep -l 'pylint\|flake8\|ruff' 2>/dev/null || echo "Static analysis tool not found"

# Discover and execute the project's validation test suite that confirms request and response models
# enforce required schema constraints
find . -type f -path '*/tests/*' -name '*validation*' -o -path '*/tests/*' -name '*schema*'

# Verify async/await usage in integration adapter files
grep -r 'async def' . --include='*.py' | grep -E '(adapter|integration)' | head -10

# Verify structured validation models are present
grep -r 'class.*Request\|class.*Response' . --include='*.py' | grep -E '(pydantic|BaseModel)' | head -10
```

**Accept when:**
- All integration adapter message delivery operations use async send methods and validation passes without blocking event loop
- HTTP API endpoints declare structured response models and validation rejects malformed requests with appropriate error codes
- Background tasks use async HTTP client sessions with documented lifecycle management and environment-based configuration
- All async message delivery methods are wrapped in try-except blocks with appropriate logging and HTTP exception handling
- Async HTTP client sessions are configured with timeout values documented in deployment configuration
- Static analysis tooling confirms no blocking calls exist within async integration adapter methods
- Integration tests validate async message delivery workflows against mock external platform endpoints

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-ASYNC-001 through R-ASYNC-007 are mandatory for integration boundary implementations requiring real-time message delivery.
</enforcement>