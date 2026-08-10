# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Internal Validate Input

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including API route handlers that coordinate with real-time message delivery systems, background task implementations that perform scheduled message transmission, service endpoints that manage reminder creation and scheduling workflows, agent state monitoring endpoints, and external platform client integrations that require asynchronous I/O operations.

### Rules

- **R-ASYNC-001** SHOULD: Internal APIs SHOULD validate input data using structured models before initiating asynchronous message delivery operations to ensure data integrity across execution boundaries.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify asynchronous message delivery behavior across API boundaries
find . -name "*test*" -o -name "*spec*" | head -5

# Locate the project's static analysis or linting configuration and execute checks
# that validate asynchronous function definitions use proper await semantics
find . -name ".eslintrc*" -o -name "pyproject.toml" -o -name "setup.cfg" | head -5

# Identify the project's dependency verification tooling and confirm all
# asynchronous runtime and external client libraries resolve to versions
# documented in the lock artifact
find . -name "package-lock.json" -o -name "poetry.lock" -o -name "Pipfile.lock" | head -5
```

**Accept when:**
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows
- Structured input validation using models is applied before initiating any asynchronous message delivery operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests, static analysis checks, and code review criteria MUST pass before accepting changes to internal API implementations with real-time message delivery requirements.
</enforcement>