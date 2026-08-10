# Adopt Asynchronous Real-Time Message Delivery for Internal API Boundaries: Implementations Structure Background

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all internal API implementations that require real-time message delivery capabilities.

## Context

- Internal API implementations require real-time message delivery to external communication platforms, coordinating between HTTP-based service endpoints and asynchronous message channels
- The system integrates reminder scheduling, agent state monitoring, and event notification workflows that span multiple execution contexts and require non-blocking I/O operations
- API route handlers expose REST endpoints for reminder management while coordinating with background tasks that perform scheduled message delivery through external platform clients
- The architecture separates HTTP request handling from real-time message transmission, requiring asynchronous coordination patterns to maintain responsiveness and handle concurrent operations

## Problem Statement

Internal APIs must coordinate synchronous HTTP request handling with asynchronous real-time message delivery to external platforms without blocking request threads, while maintaining clear boundaries between service definitions, external client interactions, and scheduled background operations. The system must support concurrent reminder scheduling, agent state queries, and event notifications across multiple execution contexts.

## Decision

1. SHOULD: API implementations SHOULD structure background tasks for scheduled operations separately from request-response cycles, using task lifecycle management to coordinate long-running message delivery workflows

## Policy Block

- SHOULD API implementations SHOULD structure background tasks for scheduled operations separately from request-response cycles, using task lifecycle management to coordinate long-running message delivery workflows

In scope:
- Internal API route handlers that coordinate with real-time message delivery systems
- Background task implementations that perform scheduled message transmission
- Service endpoints that manage reminder creation, scheduling, and deletion workflows
- Agent state monitoring endpoints that query and propagate event updates
- External platform client integrations that require asynchronous I/O operations

Out of scope:
- Synchronous HTTP-only APIs without real-time message delivery requirements
- Batch processing workflows that do not require immediate message transmission
- Static content delivery endpoints
- Read-only query APIs without external platform coordination

Exceptions:
- EXC-001: Legacy synchronous endpoints exist that cannot be migrated without breaking existing clients

## Rationale

- Evidence shows consistent use of asynchronous message delivery patterns across reminder management and bot coordination modules, with await semantics applied to channel and user message transmission operations
- The pattern separates HTTP request handling from real-time message delivery, enabling responsive API behavior while coordinating with external platform clients that require non-blocking I/O
- Detection across 2 files with 92.20% confidence demonstrates established architectural practice for internal APIs that bridge synchronous HTTP interfaces with asynchronous real-time communication boundaries
- The architecture supports concurrent operations including reminder scheduling, agent state queries, and event propagation through structured separation of request-response cycles from background task execution

## Consequences

Positive:
- API request handlers remain responsive and non-blocking, allowing concurrent request processing while background tasks handle message delivery
- Clear separation between HTTP service definitions and real-time message transmission enables independent scaling and testing of each concern
- Asynchronous coordination patterns support scheduled operations and long-running workflows without tying up request threads
- Structured input validation and response models provide type safety across execution boundaries between API handlers and message delivery operations

Negative:
- Asynchronous execution introduces complexity in error handling and debugging across execution contexts, requiring careful exception propagation and logging strategies
- Coordination between HTTP request lifecycle and background task lifecycle requires explicit state management for operations like reminder tracking and cancellation
- Testing asynchronous message delivery requires mock infrastructure for external platform clients and careful orchestration of concurrent execution paths
- Developers must understand asynchronous programming semantics and potential race conditions when coordinating between request handlers and background tasks

## Alternatives

- Synchronous blocking message delivery within request handlers (rejected)
  Rejected because: Blocking operations would tie up request threads during external platform communication, degrading API responsiveness and limiting concurrent request handling capacity. Evidence shows the system requires coordination with scheduled background tasks that cannot block HTTP responses.
  When valid: Only appropriate for low-throughput internal tools where request latency is not a concern and no scheduled operations are required
- Message queue intermediary for decoupled delivery (deferred)
  Rejected because: Not rejected but deferred. While message queues provide robust decoupling, the current evidence shows direct asynchronous client coordination is sufficient for the observed workload. Queue infrastructure adds operational complexity without clear benefit at current scale.
  When valid: Should be reconsidered when message delivery volume exceeds single-process capacity, when delivery guarantees require persistent queuing, or when multiple consumer processes need to share message delivery workload
- Webhook-based callback pattern for delivery confirmation (rejected)
  Rejected because: Webhook callbacks require additional infrastructure for receiving and routing confirmations, and the evidence shows direct asynchronous coordination provides sufficient delivery semantics for the current use cases without the complexity of bidirectional HTTP flows.
  When valid: Appropriate when external platforms require explicit delivery acknowledgment or when audit trails demand persistent confirmation records

## Risks

- Asynchronous task failures may go unnoticed if error handling and logging are insufficient, leading to silent message delivery failures
  Mitigation: Implement comprehensive logging at all asynchronous boundaries, use structured exception handling with explicit error propagation, and establish monitoring for background task health and message delivery success rates
  Owner: Engineering team
- Race conditions between API operations and background task state may cause inconsistent reminder tracking or duplicate message delivery
  Mitigation: Use explicit state management with appropriate locking or atomic operations for shared reminder state, implement idempotency keys for message delivery operations, and add integration tests covering concurrent access patterns
  Owner: Engineering team
- External platform client rate limits or transient failures may cause message delivery delays or failures without proper retry logic
  Mitigation: Implement exponential backoff retry strategies for transient failures, respect platform rate limits with appropriate throttling, and provide visibility into delivery queue depth and retry attempts through observability tooling
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Structure API route handlers to return HTTP responses immediately after validating input and initiating background operations, rather than awaiting completion of message delivery. Use response models that confirm operation acceptance rather than delivery completion.
- Organize background task lifecycle management separately from request handling, ensuring tasks can be started, monitored, and cancelled independently of HTTP request-response cycles. Use task registries or tracking structures to coordinate scheduled operations.
- Apply structured logging with correlation identifiers that span from API request through asynchronous message delivery, enabling traceability across execution boundaries. Log at key transition points including request receipt, background task initiation, message transmission attempts, and delivery confirmation.

## Continuation Context


Verify commands:
- Discover the project's test execution configuration and run integration tests that verify asynchronous message delivery behavior across API boundaries
- Locate the project's static analysis or linting configuration and execute checks that validate asynchronous function definitions use proper await semantics at message delivery call sites
- Identify the project's dependency verification tooling and confirm all asynchronous runtime and external client libraries resolve to versions documented in the lock artifact

Accept when:
- All integration tests pass demonstrating API route handlers return responses without blocking on message delivery operations
- Static analysis confirms no synchronous blocking calls exist at real-time message transmission boundaries
- Code review verifies separation between HTTP response generation and asynchronous background task execution for all reminder and notification workflows

## Enforcement

- Verified by: Automated integration tests in continuous integration pipeline that exercise asynchronous message delivery paths
- Verified by: Static analysis tooling that detects blocking operations at API boundaries
- Verified by: Code review checklist items verifying proper asynchronous patterns and error handling
- Verified by: Runtime monitoring that tracks message delivery latency and background task health
- Violation handling: CI pipeline fails if integration tests detect blocking behavior or missing await semantics at message delivery boundaries
- Violation handling: Code review blocks merge if synchronous patterns are introduced in real-time message delivery paths without documented exception approval
- Violation handling: Runtime alerts trigger when message delivery latency exceeds thresholds indicating potential blocking operations
- Violation handling: Architecture review required for any changes that introduce synchronous external platform communication in request handlers
- Exception process: Submit exception request to architecture review board documenting the specific blocking operation, performance impact assessment, and technical justification
- Exception process: Provide migration timeline if exception is for legacy code, including concrete milestones for adopting asynchronous patterns
- Exception process: Obtain approval from both technical lead and product owner acknowledging impact on API responsiveness and concurrent request capacity
- Exception process: Document approved exception in code comments with reference to approval decision and planned remediation date