# Use Asynchronous Message Sending for Real-Time Discord Communication: Error Handling Failed

These rules are ALWAYS ACTIVE for all adapter modules that interface with external real-time messaging platforms, reminder scheduling and delivery workflows, event-driven message dispatch triggered by state changes or user interactions, and thread creation and reply operations in messaging contexts.

### Rules

- **R-ASYNC-001** SHOULD: Error handling for failed asynchronous send operations SHOULD propagate exceptions to callers rather than silently suppressing failures.

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
Claude Code MUST NOT skip or defer verification. Continuous integration pipeline executes async pattern linting on every pull request. Code review checklist includes verification of async/await usage at I/O boundaries. Integration test suite validates non-blocking behavior under concurrent load. Pull requests with blocking calls in async contexts are automatically flagged and require remediation before merge.
</enforcement>