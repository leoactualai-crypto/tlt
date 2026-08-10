# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Nested Context Models

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase processes external events from multiple sources including Discord adapters, CloudEvents, timer contexts, and MCP service requests requiring structured validation
- Agent reasoning nodes, state management, photo processing workflows, and API endpoints require type-safe data contracts with runtime validation to prevent malformed data propagation
- Integration testing patterns rely on structured models to validate state transitions, API responses, and cross-boundary data flows between services
- Field-level constraints including numeric ranges, string patterns, and conditional requirements must be enforced consistently across event processing, decision-making, and external client interactions
- The system coordinates asynchronous task lifecycles, health checks, and batch operations where validation failures must be detected early to prevent cascading errors

## Problem Statement

Without standardized domain validation models, the system risks processing malformed event data, invalid agent decisions, and inconsistent API contracts across service boundaries. Type safety and constraint enforcement must be verifiable at both development time and runtime to ensure integration test reliability and prevent production data corruption.

## Decision

1. SHOULD: Nested context models SHOULD be composed using typed references to other validation models rather than untyped dictionaries

## Policy Block

- SHOULD Nested context models SHOULD be composed using typed references to other validation models rather than untyped dictionaries

In scope:
- Agent reasoning decision models with conditional field requirements based on decision type
- Event context models including CloudEvent, Discord, Timer, and generic event structures
- API request and response models for event management, experience tracking, and task submission endpoints
- Photo analysis and quality assessment output models with scored ratings
- Service status, health check, and metrics response models
- Task lifecycle and provenance tracking state models

Out of scope:
- Internal function parameter validation not crossing service boundaries
- Temporary data structures used within single-function scope
- Configuration objects loaded from environment variables or files
- Raw protocol buffers or binary serialization formats
- Database ORM models that provide their own validation layer

## Rationale

- The evidence shows 8 files consistently using BaseModel inheritance with Field descriptors for constraint declaration, establishing a proven pattern for type-safe domain validation
- Integration test patterns rely on structured model access patterns including state dictionary lookups and API response validation, requiring predictable serialization behavior
- Agent decision models demonstrate conditional field requirements where certain fields are required only for specific decision types, necessitating framework-level validation support
- Health check endpoints, metrics collection, and task status tracking require consistent response schemas across multiple service boundaries
- The pattern enables early detection of validation failures before data propagates through asynchronous task queues and event processing pipelines

## Consequences

Positive:
- Type safety and constraint enforcement at runtime prevents malformed data from propagating through agent workflows and service boundaries
- Automatic API schema generation from validation models ensures documentation stays synchronized with implementation
- Integration tests can assert against strongly-typed model instances rather than fragile dictionary key lookups
- Field-level metadata including descriptions and constraints improves developer experience and reduces integration errors

Negative:
- Validation overhead adds latency to high-throughput event processing paths, particularly for deeply nested models
- Model evolution requires careful handling of backward compatibility when adding required fields or changing constraints
- Complex conditional validation logic may require custom validators that are harder to test and maintain than simple type checks
- Serialization behavior differences between validation frameworks and native types can introduce subtle bugs in edge cases

## Alternatives

- Use dataclasses with manual validation logic in each function (rejected)
  Rejected because: Manual validation is error-prone, not reusable across service boundaries, and provides no automatic API schema generation or serialization guarantees
  When valid: For internal data structures that never cross service boundaries and have simple type requirements
- Rely on TypedDict and static type checking without runtime validation (rejected)
  Rejected because: Static typing alone cannot enforce constraints on external event data or detect validation failures at integration test time
  When valid: For configuration objects loaded once at startup from trusted sources
- Implement custom validation base class specific to this codebase (rejected)
  Rejected because: Reinventing validation infrastructure increases maintenance burden and lacks ecosystem tooling for API schema generation and serialization
  When valid: Never recommended given mature validation framework availability

## Risks

- Validation performance overhead in high-throughput event processing paths may exceed latency budgets
  Mitigation: Profile validation hot paths and consider caching validated model instances or using validation bypass flags for trusted internal events
  Owner: engineering team
- Breaking changes to model schemas during evolution may cause deserialization failures in distributed systems with version skew
  Mitigation: Implement schema versioning strategy and maintain backward compatibility for required fields using default values or optional semantics
  Owner: engineering team
- Complex conditional validation logic in models may become difficult to test comprehensively
  Mitigation: Extract complex validation rules into separate validator functions with dedicated unit tests and document conditional field requirements explicitly
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
- Define models in dedicated modules separate from business logic to enable reuse across service boundaries and facilitate schema evolution
- For models with conditional field requirements, document the decision type or context that determines which fields are required using field descriptions
- Include confidence scores and reasoning fields in decision and analysis output models to support debugging and observability in production

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock artifact, resolve the validation framework version, then locate and execute the project's type checking verification script
- Discover the project's test runner configuration, then execute integration tests that validate model serialization, constraint enforcement, and API contract compliance
- Discover the project's API documentation generation tool, then verify that all endpoint response models produce valid schema definitions

Accept when:
- All domain models representing events, decisions, API contracts, and workflow states inherit from the validation base class and declare field constraints
- Integration tests successfully validate model instances against expected constraints and serialization behavior without manual dictionary manipulation
- API endpoints declare response models and generate valid OpenAPI schemas automatically

## Enforcement

- Verified by: Static type checking in continuous integration pipeline detects missing model annotations
- Verified by: Integration test suite validates model constraint enforcement and serialization correctness
- Verified by: Code review checklist requires validation models for all new API endpoints and external event handlers
- Violation handling: Pull requests introducing unvalidated domain models crossing service boundaries are rejected in code review
- Violation handling: Runtime validation failures in production trigger error logging with model schema and invalid data for debugging
- Violation handling: API endpoints without declared response models fail OpenAPI schema generation checks in CI
- Exception process: Temporary validation bypass for performance-critical hot paths requires architecture review and documented justification
- Exception process: Legacy code migration may defer validation model adoption with explicit technical debt tracking
- Exception process: Internal-only data structures not crossing service boundaries may use simpler validation approaches with team lead approval