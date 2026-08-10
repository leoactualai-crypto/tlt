# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Runtime Configuration Service

These rules are ALWAYS ACTIVE for all HTTP API endpoints that initiate real-time message delivery to external platforms, background tasks and scheduled jobs that poll state and dispatch messages asynchronously, integration adapter modules that coordinate with external messaging platform APIs, and request and response models for reminder creation, retrieval, and deletion operations.

### Rules

- **R-ASYNC-001** MUST: Runtime configuration for service URLs and polling intervals MUST be sourced from environment variables with documented fallback defaults.
- **R-ASYNC-002** MUST: All async message delivery methods MUST be wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility.
- **R-ASYNC-003** MUST: Async HTTP client sessions MUST be configured with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures.
- **R-ASYNC-004** MUST: All message delivery operations MUST use async/await patterns without blocking calls within async functions.
- **R-ASYNC-005** MUST: HTTP API endpoints MUST declare structured response models and validation MUST reject malformed requests with appropriate error codes.
- **R-ASYNC-006** MUST: Background tasks MUST use async HTTP client sessions with documented lifecycle management and environment-based configuration.
- **R-ASYNC-007** SHOULD: Implement exponential backoff retry logic and rate limit tracking with circuit breaker patterns to prevent cascade failures from external platform API rate limits.
- **R-ASYNC-008** SHOULD: Enforce structured logging at all async boundaries and implement monitoring alerts for delivery failure rates.

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
- Environment variable names and default values are documented in deployment configuration
- All async message delivery methods include try-except blocks with appropriate logging and HTTP exception handling
- Async HTTP client sessions are configured with timeout values documented in code or configuration
- Static analysis confirms no blocking calls exist within async functions at integration boundaries
- Structured logging is present at all async boundaries with monitoring alerts configured for delivery failure rates

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail on detection of blocking calls within async integration adapter methods. Code review process MUST block merge requests that lack structured validation or response models at integration boundaries. Runtime monitoring MUST alert on message delivery failure rates exceeding defined thresholds.
</enforcement>