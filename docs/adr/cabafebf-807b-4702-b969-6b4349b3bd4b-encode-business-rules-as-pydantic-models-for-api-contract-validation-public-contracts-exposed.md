# Encode Business Rules as Pydantic Models for API Contract Validation: Public Contracts Exposed

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase processes domain events, user submissions, and service requests that require structured validation of business rules before persistence or downstream processing
- Multiple services expose public API contracts through HTTP endpoints and inter-service communication channels, requiring consistent data shape enforcement across service boundaries
- Domain models must encode constraints such as numeric ranges, required fields, default values, enumerated types, and nested object validation to prevent invalid state propagation
- The system handles asynchronous workflows, agent reasoning decisions, photo quality assessments, guild registrations, and event management where business rule violations must be caught at ingress points
- Evidence shows 11 files across MCP services, agents, adapters, and service layers consistently using declarative schema definitions with field-level constraints and type annotations

## Problem Statement

Without a standardized mechanism to encode and enforce business rules at API boundaries, services risk accepting invalid data that violates domain constraints, leading to runtime errors, inconsistent state, and defensive validation code scattered throughout business logic layers. The system requires a declarative approach to define data contracts that can be validated at ingress, serialized consistently, and serve as both runtime validators and documentation artifacts.

## Decision

1. MUST: All public API contracts exposed through HTTP endpoints or inter-service communication channels MUST be defined as declarative schema models with explicit type annotations for every field

## Policy Block

- MUST All public API contracts exposed through HTTP endpoints or inter-service communication channels MUST be defined as declarative schema models with explicit type annotations for every field

In scope:
- HTTP API request and response models for all service endpoints
- Inter-service message contracts for asynchronous communication
- Domain entity models that cross service boundaries
- Configuration models loaded from external sources
- Agent reasoning decision structures and workflow state models

Out of scope:
- Internal data structures used only within a single module or function
- Database ORM models that map directly to persistence schemas
- Temporary data transfer objects with lifetimes shorter than a single request
- Test fixtures and mock data structures

Exceptions:
- EXC-001: Performance-critical hot paths where validation overhead is measured and documented as unacceptable
- EXC-002: Legacy integration points where external systems provide pre-validated data with contractual guarantees

## Rationale

- Evidence shows 11 files consistently using schema models with field-level constraints, type annotations, and validation metadata, indicating an established pattern for encoding business rules declaratively
- The pattern appears across diverse contexts including guild registration, photo quality assessment, agent reasoning decisions, event management, and service monitoring, demonstrating broad applicability
- Schema models serve dual purposes as runtime validators and API documentation, reducing the gap between specification and implementation while preventing invalid data from entering the system
- Declarative field constraints eliminate scattered imperative validation code, centralizing business rules in type-safe definitions that can be statically analyzed and automatically enforced

## Consequences

Positive:
- Business rules are encoded once in schema definitions and automatically enforced at all API boundaries, eliminating redundant validation logic
- Invalid data is rejected at ingress points before reaching business logic, preventing error propagation and simplifying error handling in downstream layers
- Schema models serve as machine-readable API contracts that can drive code generation, documentation generation, and contract testing
- Type annotations enable static analysis tools to catch type mismatches at development time rather than runtime

Negative:
- Schema validation adds computational overhead at deserialization time, which may impact performance in high-throughput scenarios
- Complex cross-field business rules may be difficult to express declaratively, requiring custom validator functions that reduce the declarative clarity
- Schema evolution requires careful versioning to maintain backward compatibility, as field additions or constraint changes can break existing clients
- Developers must learn and maintain proficiency with the schema definition framework's constraint syntax and validation semantics

## Alternatives

- Implement imperative validation logic within service handlers using conditional statements and exception raising (rejected)
  Rejected because: Scatters validation logic across multiple layers, creates maintenance burden, lacks static analyzability, and provides no automatic documentation generation
  When valid: Never recommended for new development; acceptable only in legacy code scheduled for refactoring
- Use JSON Schema definitions stored as separate files with runtime validation against the schema documents (rejected)
  Rejected because: Separates type definitions from code, requires manual synchronization, lacks compile-time type checking, and adds complexity for nested object validation
  When valid: When integrating with external systems that mandate JSON Schema as the contract format
- Rely on database constraints and ORM validation as the primary business rule enforcement mechanism (rejected)
  Rejected because: Pushes validation to the persistence layer, allowing invalid data to propagate through business logic, and couples domain rules to database schema
  When valid: As a defense-in-depth measure complementing API-level validation, not as a replacement

## Risks

- Schema validation performance overhead becomes a bottleneck in high-throughput API endpoints, degrading response times
  Mitigation: Profile validation performance in production-like load tests, implement caching for compiled validators, and document exception process for performance-critical paths
  Owner: Engineering team with performance testing support
- Schema evolution breaks existing API clients when constraints are tightened or required fields are added without proper versioning
  Mitigation: Establish schema versioning policy, use API version prefixes, maintain backward compatibility for at least two versions, and provide migration guides
  Owner: API platform team
- Complex business rules that span multiple fields or require external data lookups cannot be expressed declaratively, leading to inconsistent validation approaches
  Mitigation: Document patterns for custom validators, provide examples of cross-field validation, and establish code review guidelines for when imperative validation is acceptable
  Owner: Architecture review board

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Define schema models in dedicated modules separate from business logic implementation, typically colocated with the service or domain boundary they represent
- Use field-level constraint metadata to encode numeric bounds, string length limits, and enumeration values rather than implementing these checks in business logic
- For datetime fields, configure serialization encoders to produce consistent wire formats across all services
- When schema models grow complex, decompose them into smaller composable models that can be reused across multiple API contracts

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the schema validation library in use; inspect its lock file to determine the exact resolved version
- Locate the project's test suite and identify test files that verify schema validation behavior; execute the test runner to confirm validation rules are enforced
- Search the codebase for API endpoint definitions and verify that request and response models are annotated with schema types; use static analysis tools to detect untyped endpoints

Accept when:
- All public API endpoints define request and response models using schema definitions with explicit type annotations and field constraints
- Schema validation tests demonstrate that invalid input is rejected with appropriate error messages before reaching business logic
- Static analysis confirms no API endpoints accept or return untyped dictionaries or generic data structures at service boundaries

## Enforcement

- Verified by: Automated static analysis in continuous integration pipelines that detect untyped API endpoints
- Verified by: Code review checklist requiring schema definitions for all new API contracts
- Verified by: Integration tests that verify validation behavior for boundary conditions and constraint violations
- Violation handling: Static analysis failures block pull request merging until schema definitions are added
- Violation handling: Code review identifies missing or incomplete schema definitions and requests changes before approval
- Violation handling: Runtime monitoring alerts on validation errors to detect schema drift or client contract violations
- Exception process: Developer documents the rationale for exception in architectural decision log with performance benchmarks or integration constraints
- Exception process: Architecture review board evaluates exception request and approves only with documented mitigation strategy
- Exception process: Approved exceptions are tracked in technical debt register with remediation timeline