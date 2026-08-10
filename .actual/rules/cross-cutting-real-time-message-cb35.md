# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Real Time Message

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including API route handlers that coordinate with real-time message delivery systems, background task implementations that perform scheduled message transmission, service endpoints that manage reminder creation and scheduling workflows, agent state monitoring endpoints, and external platform client integrations that require asynchronous I/O operations.

### Rules

- **R-ASYNC-001** MUST: Real-time message delivery implementations MUST coordinate with external platform clients through asynchronous I/O interfaces that support concurrent operations.
- **R-ASYNC-002** MUST: API route handlers MUST return HTTP responses immediately after validating input and initiating background operations, rather than awaiting completion of message delivery.
- **R-ASYNC-003** MUST: Response models MUST confirm operation acceptance rather than delivery completion.
- **R-ASYNC-004** MUST: Background task lifecycle management MUST be organized separately from request handling, ensuring tasks can be started, monitored, and cancelled independently of HTTP request-response cycles.
- **R-ASYNC-005** MUST: Task registries or tracking structures MUST be used to coordinate scheduled operations.
- **R-ASYNC-006** MUST: Structured logging with correlation identifiers MUST span from API request through asynchronous message delivery to enable traceability across execution boundaries.
- **R-ASYNC-007** MUST: Logging MUST occur at key transition points including request receipt, background task initiation, message transmission attempts, and delivery confirmation.
- **R-ASYNC-008** MUST: Comprehensive logging MUST be implemented at all asynchronous boundaries with structured exception handling and explicit error propagation.
- **R-ASYNC-009** MUST: Explicit state management with appropriate locking or atomic operations MUST be used for shared reminder state to prevent race conditions.
- **R-ASYNC-010** MUST: Idempotency keys MUST be implemented for message delivery operations.
- **R-ASYNC-011** MUST: Integration tests MUST cover concurrent access patterns for asynchronous operations.
- **R-ASYNC-012** MUST: Exponential backoff retry strategies MUST be implemented for transient failures in external platform communication.
- **R-ASYNC-013** MUST: External platform rate limits MUST be respected with appropriate throttling.
- **R-ASYNC-014** MUST: Visibility into delivery queue depth and retry attempts MUST be provided through observability tooling.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify asynchronous message delivery behavior across API boundaries
echo "Running integration tests for asynchronous message delivery..."
# (Exact command depends on project's test framework; inspect test configuration)

# Locate the project's static analysis or linting configuration and execute checks
# that validate asynchronous function definitions use proper await semantics
echo "Running static analysis for await semantics at message delivery call sites..."
# (Exact command depends on project's linter; inspect linting configuration)

# Identify the project's dependency verification tooling and confirm all
# asynchronous runtime and external client libraries resolve to versions
# documented in the lock artifact
echo "Verifying dependency lock artifact for asynchronous libraries..."
# (Exact command depends on project's build tool; inspect lock file)
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Structured logging with correlation identifiers is present across all asynchronous execution boundaries
- Error handling and retry logic are implemented for external platform communication failures
- Monitoring and observability tooling tracks message delivery latency and background task health

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests, static analysis checks, and dependency verification commands MUST be executed before accepting code changes. CI pipeline MUST fail if integration tests detect blocking behavior or missing await semantics at message delivery boundaries. Code review MUST block merge if synchronous patterns are introduced in real-time message delivery paths without documented exception approval.
</enforcement>