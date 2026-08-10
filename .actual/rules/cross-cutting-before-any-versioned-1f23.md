# Use Asynchronous Message Sending for Real-Time Discord Communication: Before Any Versioned

These rules are ALWAYS ACTIVE for all adapter modules that interface with external real-time messaging platforms, reminder scheduling and delivery workflows, event-driven message dispatch triggered by state changes or user interactions, and thread creation and reply operations in messaging contexts.

### Rules

- **R-ASYNC-001** MUST: Before using any versioned library for asynchronous messaging, discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.
- **R-ASYNC-002** MUST: Ensure the entire call chain from entry point to send is async to prevent accidental blocking; use async context managers for resource cleanup.
- **R-ASYNC-003** MUST: Add timeout parameters to all external send operations to prevent indefinite hangs; choose timeout values based on platform SLA documentation and user experience requirements.
- **R-ASYNC-004** MUST: Implement comprehensive exception handling with logging at all async send boundaries; use task exception handlers to capture and report failures.
- **R-ASYNC-005** SHOULD: Prefer structured concurrency patterns that group related tasks and propagate cancellation consistently for operations that coordinate multiple sends.
- **R-ASYNC-006** MAY: Document the technical justification for any synchronous operation in an async context and obtain approval from the architecture review board for exceptions to async patterns.

### Verify

```bash
# Discover the project's test execution script and run the integration test suite covering real-time messaging boundaries
# (Command to be derived from project repository)

# Discover the project's static analysis configuration and execute the async pattern linter to detect blocking calls in async contexts
# (Command to be derived from project repository)

# Discover the project's dependency verification script and confirm all async libraries resolve to compatible versions per the lock artifact
# (Command to be derived from project repository)
```

**Accept when:**
- All integration tests for message send operations pass without event loop blocking warnings
- Static analysis reports zero blocking I/O calls within async function bodies
- Dependency verification confirms async library versions match lock artifact and API compatibility is documented

<enforcement>
Claude Code MUST NOT skip or defer verification. All async message sending operations MUST comply with R-ASYNC-001 through R-ASYNC-006 before code review or merge.
</enforcement>