# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Implementations Expose Multiple

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including API route handlers that coordinate with real-time message delivery systems, background task implementations that perform scheduled message transmission, service endpoints that manage reminder creation and scheduling workflows, agent state monitoring endpoints, and external platform client integrations that require asynchronous I/O operations.

### Rules

- **R-ASYNC-001** MAY: Implementations MAY expose multiple HTTP methods for reminder lifecycle management including creation, retrieval, listing, and deletion operations.
- **R-ASYNC-002** MUST: API route handlers MUST return HTTP responses immediately after validating input and initiating background operations, rather than awaiting completion of message delivery.
- **R-ASYNC-003** MUST: Response models MUST confirm operation acceptance rather than delivery completion.
- **R-ASYNC-004** MUST: Background task lifecycle management MUST be organized separately from request handling, ensuring tasks can be started, monitored, and cancelled independently of HTTP request-response cycles.
- **R-ASYNC-005** MUST: Task registries or tracking structures MUST be used to coordinate scheduled operations.
- **R-ASYNC-006** MUST: Structured logging with correlation identifiers MUST span from API request through asynchronous message delivery, enabling traceability across execution boundaries.
- **R-ASYNC-007** MUST: Logging MUST occur at key transition points including request receipt, background task initiation, message transmission attempts, and delivery confirmation.
- **R-ASYNC-008** MUST: Comprehensive logging MUST be implemented at all asynchronous boundaries.
- **R-ASYNC-009** MUST: Structured exception handling with explicit error propagation MUST be established.
- **R-ASYNC-010** MUST: Monitoring for background task health and message delivery success rates MUST be established.
- **R-ASYNC-011** MUST: Explicit state management with appropriate locking or atomic operations MUST be used for shared reminder state.
- **R-ASYNC-012** MUST: Idempotency keys MUST be implemented for message delivery operations.
- **R-ASYNC-013** MUST: Integration tests covering concurrent access patterns MUST be added.
- **R-ASYNC-014** MUST: Exponential backoff retry strategies MUST be implemented for transient failures.
- **R-ASYNC-015** MUST: External platform rate limits MUST be respected with appropriate throttling.
- **R-ASYNC-016** MUST: Visibility into delivery queue depth and retry attempts MUST be provided through observability tooling.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify asynchronous message delivery behavior across API boundaries
find . -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' -o -name 'tox.ini' | head -1

# Locate the project's static analysis or linting configuration and execute checks
# that validate asynchronous function definitions use proper await semantics
find . -name '.pylintrc' -o -name 'setup.cfg' -o -name 'pyproject.toml' -o -name '.flake8' | head -1

# Identify the project's dependency verification tooling and confirm all
# asynchronous runtime and external client libraries resolve to versions
# documented in the lock artifact
find . -name 'requirements.lock' -o -name 'Pipfile.lock' -o -name 'poetry.lock' -o -name 'package-lock.json' | head -1
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Dependency verification confirms all asynchronous runtime and external client libraries resolve to documented versions

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests, static analysis checks, and dependency verification MUST pass before accepting changes to asynchronous message delivery implementations.
</enforcement>