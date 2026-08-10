# Use Asynchronous Message Sending for Real-Time Discord Communication: Components Batch Multiple

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all components that implement real-time communication boundaries with external messaging platforms.

## Context

- The system integrates with external real-time messaging platforms requiring asynchronous I/O patterns to avoid blocking event loops during message delivery operations.
- Multiple adapter components coordinate reminder scheduling, state monitoring, and event-driven message dispatch across distributed services with varying response latencies.
- The codebase uses asyncio-based concurrency primitives throughout the stack, establishing asynchronous message sending as the consistent pattern for external communication boundaries.
- Real-time user interactions demand non-blocking send operations to maintain responsiveness while coordinating with backend services for state queries and event updates.

## Problem Statement

Components that send messages to external real-time platforms must maintain system responsiveness and event-loop health while coordinating with multiple backend services. Synchronous blocking calls would degrade user experience and create cascading latency across distributed workflows.

## Decision

1. MAY: Components MAY batch multiple asynchronous send operations using task groups or gather patterns when order independence permits.

## Policy Block

- MAY Components MAY batch multiple asynchronous send operations using task groups or gather patterns when order independence permits.

In scope:
- All adapter modules that interface with external real-time messaging platforms
- Reminder scheduling and delivery workflows
- Event-driven message dispatch triggered by state changes or user interactions
- Thread creation and reply operations in messaging contexts

Out of scope:
- Synchronous logging operations
- Internal data structure updates that do not cross I/O boundaries
- Configuration loading and validation at startup
- Test fixtures and mocks that simulate message sending

Exceptions:
- EXC-001: Test code requires synchronous execution for deterministic assertion ordering

## Rationale

- The evidence shows consistent use of await patterns with send operations across both reminder and bot manager modules, indicating an established architectural boundary for asynchronous I/O.
- The system coordinates multiple backend services with varying latencies; asynchronous patterns prevent head-of-line blocking and maintain responsiveness during concurrent operations.
- The detected asyncio and aiohttp libraries in the core dependencies establish the runtime foundation for non-blocking I/O throughout the stack.
- Real-time user interactions observed in the evidence require immediate acknowledgment while background tasks complete, which asynchronous patterns naturally support.

## Consequences

Positive:
- Event loop remains responsive during message delivery, enabling concurrent handling of multiple user interactions and backend state queries.
- System can scale to handle higher message volumes without proportional increases in thread count or memory overhead.
- Consistent asynchronous patterns across adapter boundaries reduce cognitive load and simplify reasoning about concurrency.
- Non-blocking I/O enables efficient coordination of reminder scheduling, state monitoring, and message dispatch workflows.

Negative:
- Asynchronous code increases complexity for developers unfamiliar with async/await patterns and coroutine lifecycle management.
- Debugging asynchronous workflows requires specialized tooling and understanding of event loop internals.
- Mixed sync/async codebases create integration friction and require careful boundary management to avoid blocking the event loop.
- Error propagation through asynchronous call chains can obscure root causes without proper context preservation.

## Alternatives

- Use synchronous blocking send operations with thread pool executors to isolate blocking I/O (rejected)
  Rejected because: Thread pool overhead and context switching costs would degrade performance under high message volumes, and the existing asyncio-based stack would require significant refactoring to integrate thread-based concurrency safely.
  When valid: Valid only for legacy integrations with synchronous-only libraries where async wrappers are unavailable and message volumes remain low.
- Implement message queuing with background worker processes for all external sends (rejected)
  Rejected because: Adds operational complexity and latency for real-time interactions where users expect immediate feedback; the evidence shows direct send patterns without intermediate queuing infrastructure.
  When valid: Valid for batch processing workflows or when message delivery guarantees require durable persistence before acknowledgment.
- Use callback-based asynchronous patterns instead of async/await syntax (rejected)
  Rejected because: Callback-based code is harder to reason about and maintain compared to linear async/await syntax; the detected asyncio library provides native async/await support.
  When valid: Valid only when integrating with legacy callback-based libraries that lack async/await wrappers.

## Risks

- Unhandled exceptions in asynchronous send operations may silently fail or crash the event loop, causing message loss or service degradation.
  Mitigation: Implement comprehensive exception handling with logging at all async send boundaries; use task exception handlers to capture and report failures; add monitoring for send operation success rates.
  Owner: Engineering team
- Blocking operations accidentally introduced into async code paths will degrade system responsiveness and create cascading latency.
  Mitigation: Establish code review guidelines for identifying blocking calls; use static analysis tools to detect synchronous I/O in async contexts; add performance tests that measure event loop lag under load.
  Owner: Engineering team
- Version mismatches between async libraries and platform APIs may introduce incompatible behavior or deprecated patterns.
  Mitigation: Follow the lock-version grounding policy to verify API compatibility before use; maintain integration tests against the external platform's API; monitor platform changelog for breaking changes.
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
- When adding new message send operations, ensure the entire call chain from entry point to send is async to prevent accidental blocking; use async context managers for resource cleanup.
- For operations that coordinate multiple sends, prefer structured concurrency patterns that group related tasks and propagate cancellation consistently.
- Add timeout parameters to all external send operations to prevent indefinite hangs; choose timeout values based on platform SLA documentation and user experience requirements.

## Continuation Context


Verify commands:
- Discover the project's test execution script and run the integration test suite covering real-time messaging boundaries
- Discover the project's static analysis configuration and execute the async pattern linter to detect blocking calls in async contexts
- Discover the project's dependency verification script and confirm all async libraries resolve to compatible versions per the lock artifact

Accept when:
- All integration tests for message send operations pass without event loop blocking warnings
- Static analysis reports zero blocking I/O calls within async function bodies
- Dependency verification confirms async library versions match lock artifact and API compatibility is documented

## Enforcement

- Verified by: Continuous integration pipeline executes async pattern linting on every pull request
- Verified by: Code review checklist includes verification of async/await usage at I/O boundaries
- Verified by: Integration test suite validates non-blocking behavior under concurrent load
- Violation handling: Pull requests with blocking calls in async contexts are automatically flagged and require remediation before merge
- Violation handling: Runtime monitoring alerts on event loop lag exceeding defined thresholds
- Violation handling: Post-incident reviews for message delivery failures include async pattern compliance audit
- Exception process: Document the technical justification for any synchronous operation in an async context
- Exception process: Obtain approval from the architecture review board for exceptions to async patterns
- Exception process: Add inline comments explaining the exception and any mitigations for blocking behavior