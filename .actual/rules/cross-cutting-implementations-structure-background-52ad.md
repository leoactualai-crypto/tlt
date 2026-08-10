# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Implementations Structure Background

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including API route handlers that coordinate with real-time message delivery systems, background task implementations that perform scheduled message transmission, service endpoints that manage reminder creation and scheduling workflows, agent state monitoring endpoints, and external platform client integrations that require asynchronous I/O operations.

### Rules

- **R-ASYNC-001** SHOULD: API implementations SHOULD structure background tasks for scheduled operations separately from request-response cycles, using task lifecycle management to coordinate long-running message delivery workflows.
- **R-ASYNC-002** SHOULD: Structure API route handlers to return HTTP responses immediately after validating input and initiating background operations, rather than awaiting completion of message delivery. Use response models that confirm operation acceptance rather than delivery completion.
- **R-ASYNC-003** SHOULD: Organize background task lifecycle management separately from request handling, ensuring tasks can be started, monitored, and cancelled independently of HTTP request-response cycles. Use task registries or tracking structures to coordinate scheduled operations.
- **R-ASYNC-004** SHOULD: Apply structured logging with correlation identifiers that span from API request through asynchronous message delivery, enabling traceability across execution boundaries. Log at key transition points including request receipt, background task initiation, message transmission attempts, and delivery confirmation.
- **R-ASYNC-005** MUST: Implement comprehensive logging at all asynchronous boundaries, use structured exception handling with explicit error propagation, and establish monitoring for background task health and message delivery success rates to prevent silent message delivery failures.
- **R-ASYNC-006** MUST: Use explicit state management with appropriate locking or atomic operations for shared reminder state, implement idempotency keys for message delivery operations, and add integration tests covering concurrent access patterns to prevent race conditions.
- **R-ASYNC-007** MUST: Implement exponential backoff retry strategies for transient failures, respect platform rate limits with appropriate throttling, and provide visibility into delivery queue depth and retry attempts through observability tooling.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify asynchronous message delivery behavior across API boundaries
find . -name "*test*" -o -name "*spec*" | head -5

# Locate the project's static analysis or linting configuration and execute checks
# that validate asynchronous function definitions use proper await semantics
find . -name ".eslintrc*" -o -name "pyproject.toml" -o -name "setup.cfg" | head -5

# Identify the project's dependency verification tooling and confirm all
# asynchronous runtime and external client libraries resolve to versions
# documented in the lock artifact
find . -name "package-lock.json" -o -name "poetry.lock" -o -name "Pipfile.lock" | head -5
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Structured logging with correlation identifiers is present across API request through asynchronous message delivery boundaries
- Error handling and retry logic are implemented for external platform client failures
- State management uses appropriate locking or atomic operations for shared reminder state

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests, static analysis checks, and code review criteria MUST pass before accepting changes to internal API implementations with real-time message delivery requirements. CI pipeline MUST fail if blocking behavior or missing await semantics are detected at message delivery boundaries.
</enforcement>