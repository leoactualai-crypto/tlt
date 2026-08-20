# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: Agent Nodes Discord Handlers Not Use

These rules are ALWAYS ACTIVE for all agent nodes, Discord adapter handlers, service endpoints, and any component performing I/O operations in the agent or adapter layers.

### Rules

- **R-ASYNC-001** MUST_NOT: Agent nodes and Discord handlers MUST NOT use blocking synchronous I/O operations that would stall the event loop.
- **R-ASYNC-002** MUST: All classes inheriting from BaseNode MUST declare the execute method as `async def`.
- **R-ASYNC-003** MUST: All Discord command handlers, modal submission handlers, button callbacks, and select menu callbacks MUST be declared as `async def`.
- **R-ASYNC-004** MUST: All API route handlers in MCP services MUST use async/await for I/O operations.
- **R-ASYNC-005** MUST: When calling other async methods within agent nodes or handlers, MUST use `await` to properly suspend execution.
- **R-ASYNC-006** MUST: Rate limiting and delays MUST use the async sleep function from the standard library's async module, not blocking sleep.
- **R-ASYNC-007** SHOULD: When integrating synchronous libraries that perform blocking I/O, wrap calls in an executor to prevent blocking the event loop.
- **R-ASYNC-008** MAY: Pure computation functions with no I/O that complete in microseconds may remain synchronous.
- **R-ASYNC-009** MAY: Data model classes, type definitions, and utility functions for string formatting, data transformation, or validation without I/O may remain synchronous.
- **R-ASYNC-010** MAY: Configuration loading at application startup before the event loop starts may use synchronous I/O.

### Verify

```bash
# Discover and run the project's type checker to verify async method signatures
# Expected: All async method signatures match their declarations with no async/await mismatches

# Discover and run the project's linting configuration with rules detecting blocking I/O in async contexts
# Expected: No blocking I/O operations detected in async contexts

# Discover and run the project's async test suite
# Expected: All async tests execute successfully and verify concurrent execution behavior
```

**Accept when:**
- All agent node execute methods are declared as `async def` and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as `async def`
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior
- Type checking in continuous integration verifies async method signatures
- Code review checklist confirms async/await usage is correct

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures, linting violations, or test failures block acceptance. Violations require refactoring or documented exceptions with team lead review and executor thread wrapping with explicit justification.
</enforcement>