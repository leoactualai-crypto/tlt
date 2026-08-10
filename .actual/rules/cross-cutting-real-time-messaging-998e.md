# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Real Time Messaging

These rules are ALWAYS ACTIVE for all integration testing involving real-time messaging boundaries, including HTTP endpoints that trigger real-time message delivery, integration test routers exposing reminder scheduling and thread management, asynchronous operations coordinating HTTP responses with message sending, and validation models defining contracts between HTTP APIs and messaging operations.

### Rules

- **R-RTM-001** MUST: Real-time messaging operations invoked within integration test boundaries MUST use await syntax for message sending operations to ensure proper sequencing and error propagation.

### Verify

```bash
# Discover the project's test execution script or task definition and run integration tests for modules containing real-time messaging boundaries
# Inspect test output to verify that integration tests for HTTP endpoints with asynchronous messaging operations execute without timeout or event loop errors
# Examine test coverage reports to confirm that integration tests exercise both HTTP request validation and real-time message delivery code paths
```

**Accept when:**
- All integration tests for real-time messaging boundaries execute successfully with asynchronous coordination between HTTP and messaging operations
- Test coverage includes verification of message delivery, thread creation, and scheduled operation workflows across service boundaries
- No integration test failures attributed to event loop management, coroutine lifecycle, or asynchronous coordination errors

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests that fail due to synchronous coordination or missing await syntax block pull request merges until corrected. Code review must verify that new integration test endpoints use asynchronous function definitions and await syntax for messaging operations. Static analysis tools must detect synchronous function definitions in integration test modules that invoke real-time messaging clients.
</enforcement>