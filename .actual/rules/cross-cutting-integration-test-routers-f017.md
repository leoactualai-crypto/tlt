# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Integration Test Routers

These rules are ALWAYS ACTIVE for all integration testing involving real-time messaging boundaries, specifically HTTP endpoints that trigger asynchronous message delivery, reminder scheduling, and thread management operations.

### Rules

- **R-ASYNC-001** MUST: Integration test routers MUST define response models using validation frameworks to enforce contract testing between HTTP endpoints and real-time messaging operations.

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
- Integration test routers define response models that enforce the same validation rules as production endpoints
- Integration test endpoints use asynchronous function definitions and await syntax for messaging operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations block pull request merges until corrected. Code review must verify asynchronous function definitions and await syntax. Static analysis must detect synchronous function definitions in integration test modules that invoke real-time messaging clients. Exception requests require engineering team lead approval with documented timing assumptions and test flakiness mitigation strategies.
</enforcement>