# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Message Delivery Operations

These rules are ALWAYS ACTIVE for all HTTP API endpoints, background tasks, and integration adapter modules that initiate real-time message delivery to external platforms.

### Rules

- **R-MSG-001** MUST: All message delivery operations at real-time integration boundaries MUST use asynchronous send methods that return awaitable coroutines.
- **R-MSG-002** MUST: Ensure all async message delivery methods are wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility.
- **R-MSG-003** MUST: Configure async HTTP client sessions with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures.
- **R-MSG-004** MUST: Document environment variable names and default values in deployment configuration to ensure consistent runtime behavior across environments.
- **R-MSG-005** SHOULD: Implement exponential backoff retry logic and rate limit tracking with circuit breaker patterns to prevent cascade failures from external platform API rate limits.
- **R-MSG-006** SHOULD: Enforce structured logging at all async boundaries and implement monitoring alerts for delivery failure rates.

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
- All async message delivery methods include try-except blocks with appropriate logging and HTTP error responses
- Async HTTP client sessions are configured with timeout values matching external platform SLAs
- Environment variables for service URLs and polling intervals are documented

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail on detection of blocking calls within async integration adapter methods. Code review process MUST block merge requests that lack structured validation or response models at integration boundaries. Runtime monitoring MUST alert on message delivery failure rates exceeding defined thresholds.
</enforcement>