# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Integration Tests Scheduled

These rules are ALWAYS ACTIVE for all integration tests involving real-time messaging boundaries, HTTP endpoints that trigger real-time message delivery, integration test routers exposing reminder scheduling and thread management, and asynchronous operations coordinating HTTP responses with message sending.

### Rules

- **R-ASYNC-001** SHOULD: Integration tests for scheduled real-time operations SHOULD maintain in-memory state tracking to enable verification of pending operations without introducing external dependencies.

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
Claude Code MUST NOT skip or defer verification. Continuous integration pipeline executes integration tests and fails builds on test failures or timeout errors. Code review verifies that new integration test endpoints use asynchronous function definitions and await syntax for messaging operations. Static analysis tools detect synchronous function definitions in integration test modules that invoke real-time messaging clients. Integration tests that fail due to synchronous coordination or missing await syntax block pull request merges until corrected.
</enforcement>