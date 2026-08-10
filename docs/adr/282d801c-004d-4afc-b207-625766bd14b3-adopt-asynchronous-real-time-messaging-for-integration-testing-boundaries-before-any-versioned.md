# Adopt Asynchronous Real-Time Messaging for Integration Testing Boundaries: Before Any Versioned

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all integration testing involving real-time messaging boundaries.

## Context

- The codebase integrates real-time messaging systems (discord) with asynchronous HTTP services, requiring integration tests to validate message delivery, thread creation, and reminder scheduling across service boundaries.
- Integration test endpoints expose real-time messaging operations through HTTP routers, enabling verification of asynchronous workflows including message sending, thread management, and scheduled reminder delivery.
- The testing strategy must validate both synchronous HTTP request-response cycles and asynchronous real-time message delivery, requiring coordination between FastAPI routers and discord client operations.
- Evidence shows integration test endpoints defined with router.post, router.get, and router.delete decorators that invoke asynchronous real-time operations including thread.send(content) and channel.send(content).

## Problem Statement

Integration testing of real-time messaging boundaries requires a strategy that validates both HTTP API contracts and asynchronous message delivery semantics, ensuring that scheduled operations, thread creation, and message routing behave correctly across service boundaries without introducing test flakiness or timing dependencies.

## Decision

1. MUST: Before using any versioned asynchronous runtime, HTTP framework, validation library, or messaging client, the consumer MUST discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.

## Policy Block

- MUST Before using any versioned asynchronous runtime, HTTP framework, validation library, or messaging client, the consumer MUST discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.

In scope:
- HTTP endpoints that trigger real-time message delivery
- Integration test routers exposing reminder scheduling and thread management
- Asynchronous operations coordinating HTTP responses with message sending
- Validation models defining contracts between HTTP APIs and messaging operations

Out of scope:
- Unit tests that mock real-time messaging clients
- End-to-end tests running against production messaging infrastructure
- Synchronous HTTP endpoints without real-time messaging dependencies
- Performance testing of message throughput or latency

Exceptions:
- EXC-001: Integration tests require deterministic timing for scheduled operations and cannot rely on asynchronous coordination

## Rationale

- Evidence shows 2 files implementing integration test boundaries with asynchronous real-time messaging operations, including router.post and router.delete endpoints that invoke await thread.send(content) and channel.send(content), demonstrating a consistent pattern of coordinating HTTP APIs with asynchronous message delivery.
- The pattern achieves 92.20% confidence across files that define both HTTP response models and real-time messaging operations, indicating a deliberate testing strategy that validates contracts at service boundaries.
- Asynchronous coordination enables integration tests to verify end-to-end workflows including reminder scheduling, thread creation, and message routing without introducing timing dependencies or test flakiness.
- The use of validation frameworks for response models provides contract testing capabilities that ensure HTTP API changes remain compatible with real-time messaging semantics.

## Consequences

Positive:
- Integration tests can verify complete workflows spanning HTTP request handling and asynchronous message delivery in a single test execution.
- Asynchronous coordination eliminates race conditions and timing dependencies that would otherwise cause test flakiness in real-time messaging scenarios.
- Response model validation provides compile-time contract verification between HTTP APIs and messaging operations, catching integration errors early.
- In-memory state tracking enables verification of scheduled operations without requiring external infrastructure or complex test fixtures.

Negative:
- Asynchronous integration tests require careful management of event loops and coroutine lifecycle, increasing test complexity compared to synchronous approaches.
- Tests that coordinate HTTP and messaging boundaries may have longer execution times due to asynchronous operation sequencing.
- Debugging failures in asynchronous integration tests can be more challenging due to concurrent execution and non-deterministic ordering of operations.
- In-memory state tracking for scheduled operations may not accurately reflect production behavior with persistent storage or distributed scheduling.

## Alternatives

- Use synchronous integration tests with mocked real-time messaging clients (rejected)
  Rejected because: Mocking eliminates validation of asynchronous coordination semantics and message delivery behavior, reducing confidence that integration tests reflect production workflows.
  When valid: Valid for unit tests focused on HTTP request validation without real-time messaging dependencies.
- Implement end-to-end tests against production messaging infrastructure (rejected)
  Rejected because: Production infrastructure introduces external dependencies, test environment complexity, and potential for test interference with live systems, making tests slower and less reliable.
  When valid: Valid for smoke tests or deployment verification in staging environments with isolated messaging infrastructure.
- Separate HTTP API tests from real-time messaging tests with contract testing (deferred)
  Rejected because: Contract testing provides stronger isolation but requires additional tooling and coordination to ensure contracts remain synchronized across service boundaries.
  When valid: Valid when services are developed by separate teams or when integration test execution time becomes prohibitive.

## Risks

- Asynchronous integration tests may introduce non-deterministic failures if event loop management or coroutine lifecycle is not properly handled.
  Mitigation: Establish clear patterns for test fixture setup and teardown that ensure proper event loop initialization and cleanup. Use timeout mechanisms to detect hung operations.
  Owner: Engineering team
- In-memory state tracking for scheduled operations may diverge from production behavior with persistent storage, causing tests to pass while production fails.
  Mitigation: Supplement integration tests with smoke tests against staging environments that use production-equivalent persistence mechanisms. Document state tracking limitations in test documentation.
  Owner: Engineering team
- Version incompatibilities between asynchronous runtime, HTTP framework, and messaging client may cause integration test failures that do not reflect application logic errors.
  Mitigation: Enforce lock-version grounding policy to verify API compatibility before writing integration tests. Maintain compatibility matrix documentation for tested version combinations.
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
- Define integration test routers in separate modules from production routers to maintain clear boundaries between test infrastructure and application code. Use dependency injection to provide test-specific messaging clients that enable verification without external dependencies.
- Structure integration test endpoints to mirror production API contracts while exposing additional verification capabilities such as state inspection or operation cancellation. Ensure response models enforce the same validation rules as production endpoints.
- Implement test fixtures that manage event loop lifecycle and ensure proper cleanup of asynchronous resources including messaging clients, HTTP sessions, and scheduled tasks. Use context managers to guarantee cleanup even when tests fail.

## Continuation Context


Verify commands:
- Discover the project's test execution script or task definition and run integration tests for modules containing real-time messaging boundaries
- Inspect test output to verify that integration tests for HTTP endpoints with asynchronous messaging operations execute without timeout or event loop errors
- Examine test coverage reports to confirm that integration tests exercise both HTTP request validation and real-time message delivery code paths

Accept when:
- All integration tests for real-time messaging boundaries execute successfully with asynchronous coordination between HTTP and messaging operations
- Test coverage includes verification of message delivery, thread creation, and scheduled operation workflows across service boundaries
- No integration test failures attributed to event loop management, coroutine lifecycle, or asynchronous coordination errors

## Enforcement

- Verified by: Continuous integration pipeline executes integration tests and fails builds on test failures or timeout errors
- Verified by: Code review verifies that new integration test endpoints use asynchronous function definitions and await syntax for messaging operations
- Verified by: Static analysis tools detect synchronous function definitions in integration test modules that invoke real-time messaging clients
- Violation handling: Integration tests that fail due to synchronous coordination or missing await syntax block pull request merges until corrected
- Violation handling: Code review feedback requires refactoring of synchronous integration tests to use asynchronous coordination patterns
- Violation handling: Documentation of violations in test failure reports to enable root cause analysis and pattern improvement
- Exception process: Request exception approval from engineering team lead with justification for synchronous testing approach
- Exception process: Document timing assumptions, test flakiness mitigation strategies, and limitations in test module docstrings
- Exception process: Schedule follow-up work to migrate synchronous tests to asynchronous coordination when technical constraints are resolved