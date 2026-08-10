# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Before Any Versioned

These rules are ALWAYS ACTIVE for all integration testing involving real-time messaging boundaries, including HTTP endpoints that trigger real-time message delivery, integration test routers exposing reminder scheduling and thread management, and asynchronous operations coordinating HTTP responses with message sending.

### Rules

- **R-ASYNC-001** MUST: Before using any versioned asynchronous runtime, HTTP framework, validation library, or messaging client, discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.
- **R-ASYNC-002** MUST: Define integration test routers in separate modules from production routers to maintain clear boundaries between test infrastructure and application code, using dependency injection to provide test-specific messaging clients.
- **R-ASYNC-003** MUST: Structure integration test endpoints to mirror production API contracts while exposing additional verification capabilities such as state inspection or operation cancellation, ensuring response models enforce the same validation rules as production endpoints.
- **R-ASYNC-004** MUST: Implement test fixtures that manage event loop lifecycle and ensure proper cleanup of asynchronous resources including messaging clients, HTTP sessions, and scheduled tasks, using context managers to guarantee cleanup even when tests fail.
- **R-ASYNC-005** MUST: Use asynchronous function definitions and await syntax for messaging operations in integration test modules.

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
- Code review verifies that new integration test endpoints use asynchronous function definitions and await syntax for messaging operations
- Static analysis tools detect no synchronous function definitions in integration test modules that invoke real-time messaging clients

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests that fail due to synchronous coordination or missing await syntax block pull request merges until corrected. Code review feedback requires refactoring of synchronous integration tests to use asynchronous coordination patterns. Violations must be documented in test failure reports to enable root cause analysis and pattern improvement.
</enforcement>