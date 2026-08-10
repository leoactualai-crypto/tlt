# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Internal Route Handlers

These rules are ALWAYS ACTIVE for all internal API route handlers that coordinate with real-time message delivery systems, background task implementations that perform scheduled message transmission, service endpoints that manage reminder creation and scheduling workflows, agent state monitoring endpoints, and external platform client integrations requiring asynchronous I/O operations.

### Rules

- **R-ASYNC-001** MUST: Internal API route handlers MUST separate HTTP response generation from asynchronous message delivery operations, allowing immediate response to clients while background tasks handle real-time transmission.

### Verify

```bash
# Discover and run integration tests that verify asynchronous message delivery behavior across API boundaries
find . -type f -name '*test*' -o -name '*spec*' | grep -E '(integration|e2e)' | head -5

# Locate static analysis or linting configuration and execute checks validating asynchronous function definitions use proper await semantics
find . -type f \( -name '.eslintrc*' -o -name 'pylintrc' -o -name 'pyproject.toml' -o -name 'setup.cfg' \) | head -3

# Identify dependency verification tooling and confirm asynchronous runtime libraries resolve to versions in lock artifact
find . -type f \( -name 'package-lock.json' -o -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'go.sum' \) | head -1
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Dependency verification confirms all asynchronous runtime and external client libraries resolve to versions documented in the lock artifact

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests, static analysis checks, and dependency verification MUST pass before accepting changes to internal API route handlers that coordinate with real-time message delivery systems.
</enforcement>