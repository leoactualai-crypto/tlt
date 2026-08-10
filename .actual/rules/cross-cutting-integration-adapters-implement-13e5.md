# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Integration Adapters Implement

These rules are ALWAYS ACTIVE for all HTTP API endpoints that initiate real-time message delivery to external platforms, background tasks and scheduled jobs that poll state and dispatch messages asynchronously, and integration adapter modules that coordinate with external messaging platform APIs.

### Rules

- **R-ASYNC-001** SHOULD: Integration adapters SHOULD implement structured logging with named loggers to trace message delivery workflows across async boundaries.

### Verify

```bash
# Discover and execute the project's integration test suite that validates async message delivery workflows against mock external platform endpoints
# Discover and run the project's static analysis tooling to verify all message delivery operations use async/await patterns without blocking calls
# Discover and execute the project's validation test suite that confirms request and response models enforce required schema constraints
```

**Accept when:**
- All integration adapter message delivery operations use async send methods and validation passes without blocking event loop
- HTTP API endpoints declare structured response models and validation rejects malformed requests with appropriate error codes
- Background tasks use async HTTP client sessions with documented lifecycle management and environment-based configuration
- Structured logging with named loggers is present in all message delivery workflows across async boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration adapters MUST implement structured logging with named loggers. Static analysis and integration tests MUST pass before code is accepted.
</enforcement>