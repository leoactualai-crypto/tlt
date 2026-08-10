# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Real Time Message

These rules are ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities, including route handlers coordinating with external communication platforms, background task implementations performing scheduled message transmission, and service endpoints managing reminder and event notification workflows.

### Rules

- **R-ASYNC-001** MUST: All real-time message delivery operations at internal API boundaries MUST use asynchronous execution patterns with await semantics to prevent blocking request handlers.

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
- Dependency lock artifacts confirm all asynchronous runtime and external client libraries are pinned to documented versions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification steps (integration tests, static analysis, dependency verification) are mandatory before accepting changes to real-time message delivery code paths.
</enforcement>