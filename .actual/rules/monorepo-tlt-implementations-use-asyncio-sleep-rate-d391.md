# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: Implementations Use Asyncio Sleep Rate Limiting

These rules are ALWAYS ACTIVE for all agent nodes inheriting from BaseNode, all Discord command handlers, modal submission handlers, button callbacks, select menu callbacks, all API route handlers in MCP services, all message moderation enforcers and event listeners, and any component performing I/O operations in the agent or adapter layers.

### Rules

- **R-ASYNC-001** MAY: Implementations MAY use asyncio.sleep for rate limiting, delays, or simulated network latency in async contexts.
- **R-ASYNC-002** MUST: All agent node execute methods be declared as `async def`.
- **R-ASYNC-003** MUST: All Discord interaction handlers (callbacks, modal submissions, button handlers, select menu callbacks) be declared as `async def`.
- **R-ASYNC-004** MUST: Use `await` when calling other async methods within async contexts; do not fire-and-forget async calls without explicit task management.
- **R-ASYNC-005** MUST: All I/O operations in async contexts use async-compatible libraries; wrap synchronous blocking I/O in executor threads with explicit justification.
- **R-ASYNC-006** SHOULD: Avoid accidentally blocking the event loop; code review must verify that all I/O uses async libraries.
- **R-ASYNC-007** SHOULD: Wrap all async operations in try/except blocks to prevent silently swallowed exceptions.

### Verify

```bash
# Discover the project's static analysis configuration and run the type checker
# to verify async method signatures match their declarations
echo "Running type checker to verify async/await signatures..."
# (Exact command depends on project's type checker; typically: mypy, pyright, or similar)

# Discover the project's linting configuration and run the linter with rules
# that detect blocking I/O in async contexts
echo "Running linter to detect blocking I/O in async contexts..."
# (Exact command depends on project's linter; typically: pylint, flake8, or similar)

# Discover the project's test suite and run async tests to verify agent nodes
# and handlers execute without blocking
echo "Running async test suite..."
# (Exact command depends on project's test runner; typically: pytest with async plugin)
```

**Accept when:**
- All agent node execute methods are declared as `async def` and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as `async def`
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior
- Type checking in continuous integration verifies async method signatures
- Linting rules detect and flag blocking I/O in async contexts

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block pull request merge. Linting violations trigger build warnings and require justification. Code review identifies blocking operations and requests refactoring. Runtime monitoring logs event loop blocking warnings in development.
</enforcement>