# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Integration Adapters Maintain

These rules are ALWAYS ACTIVE for all HTTP API endpoints that initiate real-time message delivery to external platforms, background tasks and scheduled jobs that poll state and dispatch messages asynchronously, and integration adapter modules that coordinate with external messaging platform APIs.

### Rules

- **R-ASYNC-001** MAY: Integration adapters MAY maintain in-memory state for active delivery tasks when persistence is not required for recovery.
- **R-ASYNC-002** MUST: Ensure all async message delivery methods are wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility.
- **R-ASYNC-003** MUST: Configure async HTTP client sessions with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures.
- **R-ASYNC-004** MUST: Use async/await patterns for all message delivery operations without blocking calls within async functions.
- **R-ASYNC-005** MUST: Declare structured response models and validation that rejects malformed requests with appropriate error codes at all HTTP API endpoints.
- **R-ASYNC-006** MUST: Implement exponential backoff retry logic and rate limit tracking with circuit breaker patterns to prevent cascade failures from external platform API rate limits.
- **R-ASYNC-007** MUST: Enforce structured logging at all async boundaries and implement monitoring alerts for delivery failure rates.
- **R-ASYNC-008** SHOULD: Document environment variable names and default values in deployment configuration to ensure consistent runtime behavior across environments.
- **R-ASYNC-009** SHOULD: Evaluate persistence requirements and implement state recovery mechanisms or idempotent delivery semantics for in-memory state loss on process restart.

### Verify

```bash
# Discover and execute the project's integration test suite that validates async message delivery workflows against mock external platform endpoints
# (Command discovery required from project repository)

# Discover and run the project's static analysis tooling to verify all message delivery operations use async/await patterns without blocking calls
# (Command discovery required from project repository)

# Discover and execute the project's validation test suite that confirms request and response models enforce required schema constraints
# (Command discovery required from project repository)
```

**Accept when:**
- All integration adapter message delivery operations use async send methods and validation passes without blocking event loop
- HTTP API endpoints declare structured response models and validation rejects malformed requests with appropriate error codes
- Background tasks use async HTTP client sessions with documented lifecycle management and environment-based configuration
- All async message delivery methods include try-except blocks with exception logging and HTTP error responses
- Async HTTP client sessions are configured with appropriate timeout values
- Exponential backoff retry logic and circuit breaker patterns are implemented for external platform API interactions
- Structured logging is present at all async boundaries with monitoring alerts for delivery failures

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before accepting code changes. Rules marked SHOULD are strongly recommended and should be verified unless explicitly documented exceptions are approved through the exception process.
</enforcement>