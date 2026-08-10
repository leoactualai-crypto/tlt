# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Message Delivery Failures

These rules are ALWAYS ACTIVE for all HTTP API endpoints that initiate real-time message delivery to external platforms, background tasks and scheduled jobs that poll state and dispatch messages asynchronously, and integration adapter modules that coordinate with external messaging platform APIs.

### Rules

- **R-MSG-001** SHOULD: Message delivery failures SHOULD raise HTTP exceptions with appropriate status codes and detail messages for client error handling.

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
- All async message delivery methods are wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes
- Async HTTP client sessions are configured with timeout values appropriate for external platform SLAs

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations are detected by automated integration tests in the CI pipeline, static analysis tooling that detects blocking calls within async functions, and code review checklist items. CI pipeline fails on detection of blocking calls; code review blocks merge requests lacking structured validation or response models; runtime monitoring alerts trigger on message delivery failure rates exceeding thresholds.
</enforcement>