# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Integration Test Endpoints

These rules are ALWAYS ACTIVE for all integration test endpoints that coordinate HTTP request handling with asynchronous real-time messaging operations, including message delivery, thread creation, and reminder scheduling across service boundaries.

### Rules

- **R-ASYNC-001** SHOULD: Integration test endpoints SHOULD separate concerns between HTTP request validation, business logic execution, and real-time message delivery to enable independent verification of each boundary.

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
- Integration test routers are defined in separate modules from production routers with clear boundaries between test infrastructure and application code
- Integration test endpoints mirror production API contracts while exposing verification capabilities such as state inspection or operation cancellation
- Test fixtures properly manage event loop lifecycle and ensure cleanup of asynchronous resources including messaging clients, HTTP sessions, and scheduled tasks

<enforcement>
Claude Code MUST NOT skip or defer verification. Continuous integration pipeline MUST execute integration tests and fail builds on test failures or timeout errors. Code review MUST verify that new integration test endpoints use asynchronous function definitions and await syntax for messaging operations. Static analysis tools MUST detect synchronous function definitions in integration test modules that invoke real-time messaging clients and flag as violations.
</enforcement>