# Adopt Asynchronous Real-Time Message Delivery for Integration Boundaries: Http Endpoints That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all integration boundary implementations that require real-time message delivery.

## Context

- Integration adapters coordinate with external real-time messaging platforms using asynchronous send operations to deliver content to channels, threads, and direct message recipients
- The codebase implements HTTP API endpoints for reminder management and event handling that trigger asynchronous message delivery workflows
- Bot manager components poll external service state and dispatch pending messages using async client sessions with coordinated lifecycle management
- Runtime configuration sources control service URLs and polling intervals for state synchronization between internal services and external messaging platforms
- Pydantic models validate reminder creation requests and responses, enforcing structured data contracts at integration boundaries

## Problem Statement

Integration boundaries with real-time messaging platforms require non-blocking message delivery that coordinates with external API rate limits, connection lifecycle, and asynchronous event loops while maintaining structured validation and error handling across HTTP endpoints and background tasks.

## Decision

1. MUST: HTTP API endpoints that trigger real-time message delivery MUST declare response models that expose delivery status and identifiers

## Policy Block

- MUST HTTP API endpoints that trigger real-time message delivery MUST declare response models that expose delivery status and identifiers

In scope:
- All HTTP API endpoints that initiate real-time message delivery to external platforms
- Background tasks and scheduled jobs that poll state and dispatch messages asynchronously
- Integration adapter modules that coordinate with external messaging platform APIs
- Request and response models for reminder creation, retrieval, and deletion operations

Out of scope:
- Synchronous message delivery patterns for batch processing or offline workflows
- Internal service-to-service communication that does not cross external platform boundaries
- Database persistence layers for message history or audit logs
- Authentication and authorization mechanisms for external platform access

Exceptions:
- EX-001: Integration with legacy synchronous APIs that do not support asynchronous protocols

## Rationale

- Evidence shows consistent use of async/await patterns for message delivery across two integration adapter files with 92.20% confidence, indicating established architectural practice
- HTTP API endpoints expose structured response models and validation schemas, demonstrating commitment to contract-driven integration boundaries
- Background task coordination with async HTTP clients and environment-based configuration enables scalable state synchronization without blocking event loops
- Structured logging and exception handling patterns provide observability and error recovery capabilities essential for production real-time integrations

## Consequences

Positive:
- Non-blocking message delivery enables high-throughput integration with external platforms without saturating event loop capacity
- Structured validation at boundaries prevents malformed data from propagating into message delivery workflows
- Async HTTP client sessions with lifecycle management support connection pooling and graceful shutdown
- Environment-based configuration allows runtime adaptation to different deployment environments without code changes

Negative:
- Asynchronous control flow increases cognitive complexity for developers unfamiliar with coroutine semantics and event loop behavior
- In-memory state for active tasks introduces recovery challenges if processes restart before delivery completion
- Polling-based state synchronization may introduce latency compared to webhook or streaming approaches
- Dependency on external platform API availability creates failure modes that require circuit breaker or retry logic

## Alternatives

- Synchronous blocking message delivery with thread pool executors (rejected)
  Rejected because: Thread-based concurrency introduces higher memory overhead and context switching costs compared to async coroutines, and does not align with existing async HTTP framework adoption
  When valid: Valid only for legacy integrations with synchronous-only client libraries where async wrappers are not available
- Message queue with worker processes for decoupled delivery (deferred)
  Rejected because: Adds infrastructure complexity and operational overhead for current scale, but may be reconsidered if delivery volume exceeds single-process capacity
  When valid: Valid when message delivery throughput requirements exceed event loop capacity or when delivery guarantees require durable queuing
- Webhook-based push notifications instead of polling (deferred)
  Rejected because: Requires external platform webhook configuration and public endpoint exposure, increasing security surface area
  When valid: Valid when external platform supports reliable webhook delivery and latency requirements justify infrastructure investment

## Risks

- External platform API rate limits may cause message delivery failures or delays during traffic spikes
  Mitigation: Implement exponential backoff retry logic and rate limit tracking with circuit breaker patterns to prevent cascade failures
  Owner: engineering team
- In-memory state loss on process restart causes active delivery tasks to be orphaned without completion tracking
  Mitigation: Evaluate persistence requirements and implement state recovery mechanisms or idempotent delivery semantics
  Owner: engineering team
- Async exception handling complexity may allow silent failures in message delivery workflows
  Mitigation: Enforce structured logging at all async boundaries and implement monitoring alerts for delivery failure rates
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Ensure all async message delivery methods are wrapped in try-except blocks that log exceptions and raise HTTP exceptions with appropriate status codes for client visibility
- Configure async HTTP client sessions with timeout values appropriate for external platform SLAs to prevent indefinite blocking on network failures
- Document environment variable names and default values in deployment configuration to ensure consistent runtime behavior across environments

## Continuation Context


Verify commands:
- Discover and execute the project's integration test suite that validates async message delivery workflows against mock external platform endpoints
- Discover and run the project's static analysis tooling to verify all message delivery operations use async/await patterns without blocking calls
- Discover and execute the project's validation test suite that confirms request and response models enforce required schema constraints

Accept when:
- All integration adapter message delivery operations use async send methods and validation passes without blocking event loop
- HTTP API endpoints declare structured response models and validation rejects malformed requests with appropriate error codes
- Background tasks use async HTTP client sessions with documented lifecycle management and environment-based configuration

## Enforcement

- Verified by: Automated integration tests in continuous integration pipeline that validate async message delivery workflows
- Verified by: Static analysis tooling that detects blocking calls within async functions at integration boundaries
- Verified by: Code review checklist items that verify structured validation and error handling for all new integration endpoints
- Violation handling: CI pipeline fails on detection of blocking calls within async integration adapter methods
- Violation handling: Code review process blocks merge requests that lack structured validation or response models at integration boundaries
- Violation handling: Runtime monitoring alerts trigger on message delivery failure rates exceeding defined thresholds
- Exception process: Submit exception request to architecture review board with documented justification and migration timeline
- Exception process: Provide performance impact assessment and isolation strategy for any synchronous blocking operations
- Exception process: Document exception in architecture decision log with approval signatures and review date