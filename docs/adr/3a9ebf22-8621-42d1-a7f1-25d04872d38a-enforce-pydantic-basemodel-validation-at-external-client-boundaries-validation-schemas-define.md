# Enforce Pydantic BaseModel Validation at External Client Boundaries: Validation Schemas Define

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

## Context

- The codebase processes external events from Discord adapters, CloudEvents, HTTP clients, and MCP services, requiring consistent validation of incoming data structures before business logic execution.
- Multiple service boundaries (event managers, experience managers, photo processors, monitoring endpoints) independently define validation schemas using Pydantic BaseModel with Field constraints, indicating a distributed validation strategy.
- Agent reasoning nodes, state management, and workflow orchestration depend on strongly-typed context objects (CloudEventContext, DiscordContext, TimerContext, EventContext) to maintain data integrity across asynchronous task lifecycles.
- External client interactions span multiple protocols (HTTP POST/GET/DELETE, CloudEvents, Discord bot events) with varying payload structures, necessitating protocol-agnostic validation at ingress points.
- The pattern appears in 8 files with 91.92% confidence, demonstrating consistent adoption across adapters, services, agents, and MCP processors.

## Problem Statement

External clients submit data in diverse formats across multiple protocols, and without systematic validation at service boundaries, invalid or malformed data can propagate into business logic, causing runtime errors, state corruption, or unpredictable agent behavior. The system requires a consistent mechanism to enforce data contracts, type safety, and constraint validation before external data enters domain workflows.

## Decision

1. MUST: Validation schemas MUST define explicit field constraints including type annotations, nullability, default values, and range constraints for numeric fields.

## Policy Block

- MUST Validation schemas MUST define explicit field constraints including type annotations, nullability, default values, and range constraints for numeric fields.

In scope:
- HTTP API endpoints accepting external requests
- CloudEvent handlers processing external event streams
- Discord adapter event receivers
- MCP service input processors
- Agent task submission interfaces
- Batch processing endpoints
- Health check and monitoring endpoints that accept query parameters

Out of scope:
- Internal function calls within the same service boundary
- Database query results from trusted internal stores
- Configuration loaded from environment variables or static files
- Logging and observability data structures

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily accept unvalidated data if a validation migration plan is documented and scheduled.
- EXC-002: Debug or internal-only endpoints may skip validation if they are not exposed to external clients and are protected by authentication.

## Rationale

- The evidence shows 8 files consistently using Pydantic BaseModel with Field constraints to validate external data at service boundaries, demonstrating a proven pattern for type-safe ingress validation.
- Multiple validation schemas (AgentReasoningDecision, CloudEventContext, EventCreate, ExperienceCreate, PhotoAnalysisOutput, TaskStatusResponse) enforce constraints like ge=0.0, le=1.0 for confidence scores and quality metrics, preventing invalid values from corrupting business logic.
- The pattern separates validation concerns from business logic by declaring schemas as standalone classes, enabling reuse across endpoints and maintaining clear separation between data contracts and domain workflows.
- Automatic validation by web frameworks (FastAPI response_model declarations) and schema frameworks reduces boilerplate error handling and ensures consistent validation behavior across all external client interactions.

## Consequences

Positive:
- Type safety and constraint enforcement at service boundaries prevent invalid data from propagating into business logic, reducing runtime errors and state corruption.
- Self-documenting schemas serve as explicit API contracts, improving external client integration and reducing integration errors.
- Automatic validation by frameworks eliminates manual validation boilerplate, reducing code duplication and maintenance burden.
- Structured validation errors provide actionable feedback to external clients, improving debuggability and client experience.

Negative:
- Schema definitions add upfront development overhead and require maintenance when API contracts evolve.
- Strict validation may reject edge-case inputs that could be handled gracefully, requiring careful constraint design to balance safety and flexibility.
- Schema validation introduces runtime overhead for deserialization and constraint checking, though typically negligible compared to I/O costs.
- Breaking changes to validation schemas require coordinated updates across external clients, increasing deployment coordination complexity.

## Alternatives

- Manual validation using conditional checks and type assertions within endpoint handlers (rejected)
  Rejected because: Manual validation scatters validation logic across handlers, increases code duplication, lacks consistency, and is error-prone compared to declarative schema-based validation.
  When valid: Only acceptable for trivial endpoints with single primitive parameters where schema overhead exceeds benefit
- Runtime type checking without constraint validation (rejected)
  Rejected because: Type checking alone does not enforce business constraints like value ranges, string formats, or required field combinations, allowing invalid-but-well-typed data to enter business logic.
  When valid: Acceptable for internal interfaces where business constraints are enforced by upstream components
- Validation at database layer using database constraints (rejected)
  Rejected because: Database validation occurs too late in the request lifecycle, after business logic has potentially acted on invalid data, and provides poor error feedback to external clients.
  When valid: Database constraints should complement but not replace ingress validation as a defense-in-depth measure

## Risks

- Schema evolution may introduce breaking changes that cause external client failures if not coordinated through versioning or backward-compatible changes.
  Mitigation: Implement API versioning strategy, use optional fields with defaults for additive changes, and maintain deprecation periods for field removals.
  Owner: Engineering team
- Overly strict validation constraints may reject legitimate edge-case inputs, causing false negatives and poor client experience.
  Mitigation: Design constraints based on actual business requirements rather than assumed limits, and implement monitoring to detect validation rejection patterns.
  Owner: Engineering team
- Validation framework vulnerabilities or bugs could allow malformed data to bypass validation or cause denial-of-service through expensive validation operations.
  Mitigation: Keep validation framework dependencies updated, implement input size limits, and monitor validation performance metrics.
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
- Define validation schemas as separate classes that inherit from the schema validation framework's base class, using field constraint declarations to enforce type, nullability, defaults, and range constraints.
- For HTTP endpoints, declare request body and response models using the web framework's type annotation system to enable automatic validation and serialization.
- For event processing workflows, validate incoming event payloads by instantiating the appropriate context schema class and catching validation exceptions to handle malformed events.
- Include descriptive field documentation in schema definitions to generate self-documenting API contracts and improve external client integration experience.

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the schema validation framework. Inspect the lock file to determine the exact resolved version.
- Locate validation schema definitions in the codebase and verify they inherit from the framework's base class with explicit field constraints.
- Identify HTTP endpoint handlers and verify they declare typed request and response models that are automatically validated by the web framework.
- Locate event processing workflows and verify they instantiate validation schemas before passing data to business logic or state management.

Accept when:
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.

## Enforcement

- Verified by: Code review verification that all new external client endpoints define validation schemas with explicit constraints.
- Verified by: Static analysis tooling to detect unvalidated external data flows into business logic.
- Verified by: Integration tests that submit invalid payloads and verify validation errors are returned with structured error messages.
- Violation handling: Code review rejection for new endpoints that accept external data without schema validation.
- Violation handling: Static analysis warnings escalated to errors in CI pipeline for unvalidated external data flows.
- Violation handling: Post-deployment monitoring alerts for unexpected validation error rates indicating schema mismatches with external clients.
- Exception process: Exception requests must document the specific endpoint, justification for skipping validation, and mitigation plan for data integrity risks.
- Exception process: Engineering lead approval required with security review for endpoints exposed to external clients.
- Exception process: Exceptions must be time-bound with documented migration plan to implement validation within agreed timeline.