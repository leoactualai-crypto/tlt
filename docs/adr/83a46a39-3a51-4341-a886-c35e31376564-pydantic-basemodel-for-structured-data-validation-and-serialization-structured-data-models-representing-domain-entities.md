# Pydantic BaseModel for Structured Data Validation and Serialization: Structured Data Models Representing Domain Entities

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The project spans multiple services (MCP services, agent nodes, Discord adapters) that exchange structured data across boundaries and require runtime validation guarantees
- Domain models, API contracts, and configuration schemas need automatic JSON serialization with custom type handling (datetime, complex nested objects)
- Type safety at development time (via type hints) must be enforced at runtime to catch data integrity issues before they propagate through the system
- Service layer components require self-documenting data contracts that serve as both validation logic and API documentation

## Problem Statement

Services exchanging structured data across boundaries need runtime type validation, automatic serialization, and self-documenting contracts. Without a validation framework, manual type checking is error-prone, serialization logic scatters across modules, and data integrity issues propagate silently through service layers.

## Decision

1. MUST: All structured data models representing domain entities, API contracts, configuration schemas, or data transfer objects MUST inherit from Pydantic BaseModel to enforce runtime type validation and enable automatic serialization

## Policy Block

- MUST All structured data models representing domain entities, API contracts, configuration schemas, or data transfer objects MUST inherit from Pydantic BaseModel to enforce runtime type validation and enable automatic serialization

In scope:
- Domain models representing business entities (Guild, Event, User, RSVP)
- API request and response models for service endpoints
- Configuration schemas for service settings and feature flags
- Data transfer objects passed between agent nodes or service layers
- State models for agent workflows requiring validation

Out of scope:
- Simple data structures used only within a single function scope
- Internal utility classes not crossing module boundaries
- Performance-critical hot paths where validation overhead is measured and unacceptable
- Legacy code interfacing with systems that cannot consume Pydantic models

## Rationale

- Evidence shows Pydantic BaseModel usage across 6 files spanning guild management, event management, agent state, and Discord adapters with pattern significance 0.89-0.91, indicating established architectural practice
- The pattern provides runtime validation that catches type errors at data ingestion points rather than allowing invalid data to propagate through service layers
- Automatic JSON serialization with custom encoders (datetime.isoformat()) eliminates repetitive serialization logic and ensures consistent API response formats
- Type-annotated models serve dual purpose as validation logic and living documentation for API contracts and domain entities

## Consequences

Positive:
- Runtime type validation catches data integrity issues at service boundaries before they propagate
- Automatic JSON serialization reduces boilerplate and ensures consistent datetime/enum handling across APIs
- Type hints enable IDE autocomplete and static analysis while Pydantic enforces them at runtime
- Self-documenting models reduce need for separate API documentation and improve developer onboarding

Negative:
- Runtime validation adds computational overhead to model instantiation, potentially impacting high-throughput endpoints
- Pydantic version upgrades may introduce breaking changes to validation behavior or Config syntax
- Complex nested models with many validators can make debugging validation errors difficult
- Tight coupling to Pydantic makes migration to alternative validation frameworks costly

## Alternatives

- Use Python dataclasses with manual validation logic (rejected)
  Rejected because: Dataclasses provide structure but no runtime validation; manual validation logic would scatter across modules and lack consistency
  When valid: Simple internal data structures not requiring validation or serialization
- Use marshmallow for serialization and validation (rejected)
  Rejected because: Marshmallow separates schema from data objects, requiring more boilerplate; Pydantic's model-centric approach is more ergonomic for this codebase
  When valid: Projects requiring strict separation between data objects and validation schemas
- Use attrs with validators (rejected)
  Rejected because: Attrs focuses on class definition; Pydantic provides superior JSON serialization and validation error messages out of the box
  When valid: Projects prioritizing class definition ergonomics over serialization features

## Risks

- Pydantic validation overhead degrades performance in high-throughput API endpoints processing thousands of requests per second
  Mitigation: Profile critical paths, measure validation overhead, and selectively disable validation or use simpler models for performance-critical hot paths (see policy_scope_out)
  Owner: Backend engineering team
- Breaking changes in Pydantic major version upgrades (v1 to v2) require extensive model refactoring across the codebase
  Mitigation: Pin Pydantic major version in dependency manifest, test upgrades in isolated branch, maintain compatibility layer during migration
  Owner: Platform engineering team
- Complex validation errors from deeply nested models are difficult to debug and expose confusing messages to API consumers
  Mitigation: Implement custom error handlers that transform Pydantic ValidationError into user-friendly API error responses with field-level details
  Owner: API engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When defining models with datetime fields that need JSON serialization, use Field(default_factory=lambda: datetime.now(timezone.utc)) for auto-timestamps and define Config.json_encoders to map datetime to isoformat()
- For optional fields with complex defaults (dicts, lists), always use Field(default_factory=dict) or Field(default_factory=list) to avoid shared mutable default issues
- Import BaseModel and Field from the validation library's root namespace; verify the exact import paths and available Field parameters in the locked version's documentation before use

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock artifact; identify the validation library version; confirm BaseModel and Field are available in that version
- Discover the project's test suite location; locate and execute model validation tests that instantiate models with valid and invalid data
- Discover the project's static analysis configuration; execute type checking to verify all model fields have explicit type annotations

Accept when:
- All structured data models in domain, API, and configuration modules inherit from the validation library's BaseModel class
- Model instantiation with invalid data raises validation errors at runtime, preventing invalid data from entering the system
- JSON serialization of models with datetime fields produces ISO 8601 formatted strings without manual conversion logic

## Enforcement

- Verified by: Code review checklist verifies new data models inherit from BaseModel and include type annotations
- Verified by: Static type checking in CI pipeline validates type annotation completeness
- Verified by: Integration tests verify model validation behavior and JSON serialization correctness
- Violation handling: Code review blocks merge if structured data models lack BaseModel inheritance or type annotations
- Violation handling: CI pipeline fails if static type checking detects missing annotations on model fields
- Violation handling: Runtime validation errors in production trigger alerts for investigation of data integrity issues
- Exception process: Performance-critical hot paths may request exception via architecture review with profiling data demonstrating validation overhead impact
- Exception process: Legacy integration points may use alternative serialization temporarily with documented migration plan
- Exception process: Exceptions require explicit documentation in code comments explaining rationale and expected migration timeline