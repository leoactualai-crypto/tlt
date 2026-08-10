# Use Asynchronous Message Sending for Real-Time Discord Communication: Message Send Operations

These rules are ALWAYS ACTIVE for all adapter modules that interface with external real-time messaging platforms, reminder scheduling and delivery workflows, event-driven message dispatch triggered by state changes or user interactions, and thread creation and reply operations in messaging contexts.

### Rules

- **R-MSG-001** MUST: All message send operations to external real-time messaging platforms MUST use asynchronous await patterns to prevent event loop blocking.
- **R-MSG-002** MUST: Ensure the entire call chain from entry point to send is async to prevent accidental blocking; use async context managers for resource cleanup.
- **R-MSG-003** MUST: Add timeout parameters to all external send operations to prevent indefinite hangs; choose timeout values based on platform SLA documentation and user experience requirements.
- **R-MSG-004** SHOULD: For operations that coordinate multiple sends, prefer structured concurrency patterns that group related tasks and propagate cancellation consistently.
- **R-MSG-005** MUST: Implement comprehensive exception handling with logging at all async send boundaries; use task exception handlers to capture and report failures; add monitoring for send operation success rates.

### Verify

```bash
# Discover the project's test execution script and run the integration test suite covering real-time messaging boundaries
# (Exact command derived from project repository)

# Discover the project's static analysis configuration and execute the async pattern linter to detect blocking calls in async contexts
# (Exact command derived from project repository)

# Discover the project's dependency verification script and confirm all async libraries resolve to compatible versions per the lock artifact
# (Exact command derived from project repository)
```

**Accept when:**
- All integration tests for message send operations pass without event loop blocking warnings
- Static analysis reports zero blocking I/O calls within async function bodies
- Dependency verification confirms async library versions match lock artifact and API compatibility is documented

<enforcement>
Claude Code MUST NOT skip or defer verification. All message send operations must be audited for async/await compliance before acceptance. Blocking calls in async contexts are violations requiring remediation.
</enforcement>