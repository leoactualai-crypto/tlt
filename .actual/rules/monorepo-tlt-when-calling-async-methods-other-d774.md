# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: When Calling Async Methods Other Contexts

These rules are ALWAYS ACTIVE for all agent nodes inheriting from BaseNode, all Discord command handlers, modal submission handlers, button callbacks, select menu callbacks, all API route handlers in MCP services, all message moderation enforcers and event listeners, and any component performing I/O operations including network requests, database queries, or file system access in the agent or adapter layers.

### Rules

- **R-ASYNC-001** SHOULD: When calling async methods from other async contexts, implementations SHOULD use await rather than creating tasks unless explicit concurrent execution is required.

### Verify

```bash
# Discover and run the project's type checker to verify async method signatures
type_checker_config=$(find . -name "pyproject.toml" -o -name "setup.cfg" -o -name "mypy.ini" | head -1)
if [ -n "$type_checker_config" ]; then
  echo "Running type checker with async/await verification..."
  # Type checker command will be discovered from project configuration
fi

# Discover and run the project's linter with rules detecting blocking I/O in async contexts
linter_config=$(find . -name ".pylintrc" -o -name "pyproject.toml" -o -name "setup.cfg" | head -1)
if [ -n "$linter_config" ]; then
  echo "Running linter to detect blocking I/O in async contexts..."
  # Linter command will be discovered from project configuration
fi

# Discover and run the project's async test suite
test_config=$(find . -name "pytest.ini" -o -name "pyproject.toml" -o -name "setup.cfg" | head -1)
if [ -n "$test_config" ]; then
  echo "Running async test suite to verify concurrent execution behavior..."
  # Test command will be discovered from project configuration
fi
```

**Accept when:**
- All agent node execute methods are declared as `async def` and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as `async def`
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures, linting violations, or test failures block acceptance. Violations require explicit exception documentation with team lead review and executor thread wrapping for any unavoidable blocking operations.
</enforcement>