# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Before Implementing Any

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including API route handlers coordinating with message delivery systems, background task implementations performing scheduled transmission, service endpoints managing reminder workflows, agent state monitoring endpoints, and external platform client integrations requiring asynchronous I/O operations.

### Rules

- **R-ASYNC-001** MUST: Before implementing any asynchronous runtime or external client library, discover the project's dependency lock artifact, resolve the exact installed version, and verify all async APIs against that version's official documentation.
- **R-ASYNC-002** MUST: Structure API route handlers to return HTTP responses immediately after validating input and initiating background operations, rather than awaiting completion of message delivery.
- **R-ASYNC-003** MUST: Use response models that confirm operation acceptance rather than delivery completion.
- **R-ASYNC-004** MUST: Organize background task lifecycle management separately from request handling, ensuring tasks can be started, monitored, and cancelled independently of HTTP request-response cycles.
- **R-ASYNC-005** MUST: Use task registries or tracking structures to coordinate scheduled operations.
- **R-ASYNC-006** MUST: Apply structured logging with correlation identifiers that span from API request through asynchronous message delivery.
- **R-ASYNC-007** MUST: Log at key transition points including request receipt, background task initiation, message transmission attempts, and delivery confirmation.
- **R-ASYNC-008** MUST: Implement comprehensive logging at all asynchronous boundaries with structured exception handling and explicit error propagation.
- **R-ASYNC-009** MUST: Use explicit state management with appropriate locking or atomic operations for shared reminder state.
- **R-ASYNC-010** MUST: Implement idempotency keys for message delivery operations.
- **R-ASYNC-011** MUST: Implement exponential backoff retry strategies for transient failures.
- **R-ASYNC-012** MUST: Respect platform rate limits with appropriate throttling.
- **R-ASYNC-013** MUST: Provide visibility into delivery queue depth and retry attempts through observability tooling.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify asynchronous message delivery behavior across API boundaries
find . -name "pytest.ini" -o -name "setup.cfg" -o -name "tox.ini" -o -name "pyproject.toml" | head -1
# Then execute: pytest -v --tb=short <test_directory>

# Locate the project's static analysis or linting configuration and execute checks
# that validate asynchronous function definitions use proper await semantics
find . -name ".pylintrc" -o -name "pyproject.toml" -o -name "setup.cfg" | head -1
# Then execute: pylint --disable=all --enable=W1113,W1114 <source_directory>

# Identify the project's dependency verification tooling and confirm all
# asynchronous runtime and external client libraries resolve to versions
# documented in the lock artifact
find . -name "requirements.lock" -o -name "Pipfile.lock" -o -name "poetry.lock" -o -name "package-lock.json" | head -1
# Then verify installed versions match lock artifact
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Dependency lock artifact versions are verified against official documentation before implementation
- Structured logging with correlation identifiers is present across all asynchronous boundaries
- Error handling and retry logic are implemented for external platform client interactions

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-ASYNC rules are mandatory before implementing asynchronous message delivery in internal API boundaries. Violations must be escalated to architecture review with documented justification and migration timeline.
</enforcement>