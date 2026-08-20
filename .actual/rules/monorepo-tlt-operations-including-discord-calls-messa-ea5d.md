# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: Operations Including Discord Calls Message Fetching

These rules are ALWAYS ACTIVE for all agent nodes, Discord adapter handlers, service endpoints, and any component performing I/O operations in the agent or adapter layers.

### Rules

- **R-ASYNC-001** MUST: All I/O operations including Discord API calls, message fetching, external service requests, and sleep operations MUST use await with async-compatible libraries.
- **R-ASYNC-002** MUST: All classes inheriting from BaseNode in the agent node architecture MUST declare the execute method as `async def`.
- **R-ASYNC-003** MUST: All Discord command handlers, modal submission handlers, button callbacks, and select menu callbacks MUST be declared as `async def`.
- **R-ASYNC-004** MUST: All API route handlers in MCP services MUST use async/await for I/O operations.
- **R-ASYNC-005** MUST: When introducing delays or rate limiting, use the async sleep function from the standard library's async module rather than blocking sleep.
- **R-ASYNC-006** SHOULD: If integrating a synchronous library that performs blocking I/O, wrap calls in an executor to prevent blocking the event loop.
- **R-ASYNC-007** SHOULD: All async method calls MUST use await rather than fire-and-forget tasks.
- **R-ASYNC-008** SHOULD: Exception handling MUST wrap async operations in try/except blocks to prevent silently swallowed exceptions.

### Verify

```bash
# Discover and run the project's type checker to verify async method signatures
# Expected: No async/await mismatches reported

# Discover and run the project's linter with rules detecting blocking I/O in async contexts
# Expected: No blocking I/O violations in async code paths

# Discover and run the project's async test suite
# Expected: All async tests pass and concurrent execution behavior is validated
```

**Accept when:**
- All agent node execute methods are declared as `async def` and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as `async def`
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures, linting violations, or test failures block acceptance.
</enforcement>