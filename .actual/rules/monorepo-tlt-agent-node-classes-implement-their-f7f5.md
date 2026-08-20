# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: Agent Node Classes Implement Their Execute

These rules are ALWAYS ACTIVE for all agent node classes, Discord adapter handlers, service endpoints, and any component performing I/O operations in the agent or adapter layers.

### Rules

- **R-ASYNC-001** MUST: All agent node classes MUST implement their execute method as `async def` and return an awaitable that resolves to the updated state.
- **R-ASYNC-002** MUST: All Discord command handlers, modal submission handlers, button callbacks, and select menu callbacks MUST be declared as `async def`.
- **R-ASYNC-003** MUST: All API route handlers in MCP services MUST be declared as `async def`.
- **R-ASYNC-004** MUST: All message moderation enforcers and event listeners MUST be declared as `async def`.
- **R-ASYNC-005** MUST: When calling other async methods within async contexts, MUST use `await` to prevent fire-and-forget execution.
- **R-ASYNC-006** MUST: When introducing delays or rate limiting in async contexts, MUST use the async sleep function from the standard library rather than blocking sleep.
- **R-ASYNC-007** MUST: If integrating a synchronous library that performs blocking I/O, MUST wrap calls in an executor to prevent blocking the event loop.
- **R-ASYNC-008** SHOULD: All async method calls SHOULD be wrapped in try/except blocks to ensure exceptions are properly handled and not silently swallowed.
- **R-ASYNC-009** MAY: Pure computation functions with no I/O that complete in microseconds MAY remain synchronous.
- **R-ASYNC-010** MAY: Data model classes, type definitions, and utility functions for string formatting, data transformation, or validation that do not perform I/O MAY remain synchronous.
- **R-ASYNC-011** MAY: Configuration loading at application startup before the event loop starts MAY remain synchronous.

### Verify

```bash
# Discover and run the project's type checker to verify async method signatures
# Expected: All agent node execute methods and Discord handlers are declared as async def
# Expected: No async/await mismatches detected

# Discover and run the project's linter with rules that detect blocking I/O in async contexts
# Expected: No blocking I/O operations detected in async functions

# Discover and run the project's async test suite
# Expected: All async tests pass
# Expected: Concurrent execution behavior is verified
```

**Accept when:**
- All agent node execute methods are declared as `async def` and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as `async def`
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior
- Type checking in continuous integration verifies async method signatures match their declarations

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block pull request merge. Linting violations trigger build warnings and require justification. Code review identifies blocking operations and requests refactoring. Runtime monitoring logs event loop blocking warnings in development.
</enforcement>