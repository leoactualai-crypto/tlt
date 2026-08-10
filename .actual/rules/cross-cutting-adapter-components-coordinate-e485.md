# Use Asynchronous Message Sending for Real-Time Discord Communication: Adapter Components Coordinate

These rules are ALWAYS ACTIVE for all adapter components that implement real-time communication boundaries with external messaging platforms, including reminder scheduling, state monitoring, and event-driven message dispatch workflows.

### Rules

- **R-ASYNC-001** SHOULD: Adapter components SHOULD coordinate asynchronous message sends with backend state queries using concurrent task patterns to minimize total latency.

### Verify

```bash
# Discover the project's test execution script and run the integration test suite covering real-time messaging boundaries
# Discover the project's static analysis configuration and execute the async pattern linter to detect blocking calls in async contexts
# Discover the project's dependency verification script and confirm all async libraries resolve to compatible versions per the lock artifact
```

**Accept when:**
- All integration tests for message send operations pass without event loop blocking warnings
- Static analysis reports zero blocking I/O calls within async function bodies
- Dependency verification confirms async library versions match lock artifact and API compatibility is documented

<enforcement>
Claude Code MUST NOT skip or defer verification. All adapter components sending messages to external real-time platforms must comply with these asynchronous patterns to maintain system responsiveness and event-loop health.
</enforcement>