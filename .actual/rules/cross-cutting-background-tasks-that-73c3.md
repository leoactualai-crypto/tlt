# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Background Tasks That

These rules are ALWAYS ACTIVE for all HTTP API endpoints, background tasks, scheduled jobs, and integration adapter modules that initiate or coordinate real-time message delivery to external messaging platforms.

### Rules

- **R-ASYNC-001** MUST: Background tasks that poll external service state and dispatch messages MUST use async HTTP client sessions with explicit lifecycle management.
- **R-ASYNC-002** MUST: All async message delivery methods MUST be wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility.
- **R-ASYNC-003** MUST: Async HTTP client sessions MUST be configured with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures.
- **R-ASYNC-004** MUST: All HTTP API endpoints that initiate real-time message delivery MUST declare structured response models and validation schemas.
- **R-ASYNC-005** MUST: Request and response models for reminder creation, retrieval, and deletion operations MUST enforce structured data contracts using Pydantic validation at integration boundaries.
- **R-ASYNC-006** SHOULD: Environment variable names and default values for runtime configuration MUST be documented in deployment configuration to ensure consistent behavior across environments.
- **R-ASYNC-007** SHOULD: Implement exponential backoff retry logic and rate limit tracking with circuit breaker patterns to prevent cascade failures from external platform API rate limits.
- **R-ASYNC-008** SHOULD: Enforce structured logging at all async boundaries and implement monitoring alerts for delivery failure rates.

### Verify

```bash
# Discover and execute the project's integration test suite that validates async message delivery workflows
# against mock external platform endpoints
find . -type f -name '*test*integration*' -o -name '*integration*test*' | head -5

# Discover and run the project's static analysis tooling to verify all message delivery operations
# use async/await patterns without blocking calls
grep -r "async def" --include="*.py" | grep -E "(send|deliver|dispatch)" | head -10

# Discover and execute the project's validation test suite that confirms request and response models
# enforce required schema constraints
find . -type f -name '*test*validation*' -o -name '*validation*test*' | head -5

# Verify no blocking calls within async functions at integration boundaries
grep -r "requests\." --include="*.py" | grep -v "#" | head -10
```

**Accept when:**
- All integration adapter message delivery operations use async send methods and validation passes without blocking event loop
- HTTP API endpoints declare structured response models and validation rejects malformed requests with appropriate error codes
- Background tasks use async HTTP client sessions with documented lifecycle management and environment-based configuration
- All async message delivery methods are wrapped in try-except blocks with structured logging
- Async HTTP client sessions are configured with explicit timeout values
- No blocking calls (e.g., synchronous `requests` library) are detected within async integration adapter methods

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for integration boundary implementations requiring real-time message delivery. Violations detected by static analysis or integration tests MUST block code merge.
</enforcement>