# Enforce Pydantic BaseModel for Public API Contract Validation: Optional Fields Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase exposes multiple public-facing APIs through FastAPI routers, MCP services, and agent interfaces that require structured request/response validation
- Eleven files across guild management, photo processing, event management, monitoring, and agent reasoning demonstrate consistent use of Pydantic BaseModel subclasses for domain validation
- Public API contracts require runtime type checking, field constraints (ge, le, Field descriptors), default factories, and JSON serialization for datetime and complex types
- External clients interact with these APIs through HTTP endpoints, CloudEvents, and MCP tool requests, necessitating defensive validation at service boundaries
- The pattern emerged organically across independent services (guild_manager, photo_vibe_check, tlt_service, discord_adapter) indicating a project-wide architectural preference

## Problem Statement

Public API endpoints lack consistent validation mechanisms, leading to runtime errors from malformed requests, type mismatches, and missing required fields. Without structured validation at service boundaries, invalid data propagates into business logic, causing cascading failures and poor error messages for API consumers.

## Decision

1. MUST: Optional fields MUST use Optional type annotation and provide default values or default_factory functions

## Policy Block

- MUST Optional fields MUST use Optional type annotation and provide default values or default_factory functions

In scope:
- All FastAPI router endpoint request and response models
- MCP service tool request and response schemas
- Agent reasoning decision models and state representations
- CloudEvent data payloads and context structures
- Domain entities exposed through public service interfaces

Out of scope:
- Internal domain models not exposed through public APIs
- Database ORM models and persistence layer schemas
- Private utility functions and internal data transformations
- Test fixtures and mock data structures

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily use dict-based validation
- EXC-002: Performance-critical internal APIs with trusted callers may skip validation

## Rationale

- Evidence shows 11 files with 91.85% confidence implementing BaseModel subclasses for domain validation, indicating strong project-wide adoption
- Pydantic provides runtime type checking, automatic validation, and JSON schema generation that aligns with FastAPI's native integration for API documentation
- Field constraints (ge=0.0, le=1.0) observed in PhotoAnalysis and AgentReasoningDecision models prevent invalid data at ingress points
- Consistent use of default_factory for mutable defaults (Dict, List) across GuildRegistrationData, EventContext, and PhotoSubmission models demonstrates mature understanding of Python mutability pitfalls

## Consequences

Positive:
- API consumers receive clear validation errors with field-level detail when submitting malformed requests
- FastAPI automatically generates OpenAPI schemas from BaseModel definitions, improving API discoverability and client code generation
- Type safety at service boundaries catches integration errors during development rather than production
- Consistent validation patterns reduce cognitive load when working across multiple services

Negative:
- Pydantic validation adds runtime overhead for request/response processing, measurable in high-throughput scenarios
- Model definitions require more boilerplate than simple dict structures, increasing initial development time
- Version compatibility between Pydantic major versions requires migration effort when upgrading dependencies
- Complex validation logic may require custom validators, increasing model complexity

## Alternatives

- Use dataclasses with manual validation functions (rejected)
  Rejected because: Lacks runtime validation, JSON schema generation, and FastAPI integration; requires manual validation code duplication across endpoints
  When valid: Internal models where validation is handled by upstream callers
- Use dict-based schemas with JSON Schema validation libraries (rejected)
  Rejected because: Separates type definitions from validation logic; no static type checking; poor IDE support for autocomplete and refactoring
  When valid: Configuration files or external schema definitions where Python types are not primary source of truth
- Use attrs library with validators (rejected)
  Rejected because: Lacks native FastAPI integration; requires custom serialization for datetime and complex types; smaller ecosystem than Pydantic
  When valid: Projects not using FastAPI or requiring more control over validation semantics

## Risks

- Pydantic major version upgrades may break existing models due to API changes between v1 and v2
  Mitigation: Pin Pydantic major version in dependency manifest; establish migration testing protocol before upgrading; monitor Pydantic changelog for breaking changes
  Owner: Engineering team
- Complex nested models with deep validation may introduce performance bottlenecks in high-frequency endpoints
  Mitigation: Profile validation overhead in performance-critical paths; consider validation caching or trusted caller exceptions for internal APIs; monitor request latency metrics
  Owner: Engineering team
- Inconsistent Field constraint patterns across services may lead to divergent validation behavior for similar domain concepts
  Mitigation: Establish shared model library for common domain types; document standard constraint patterns in architecture guidelines; conduct periodic validation audits
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
- When defining models with datetime fields, include Config class with json_encoders mapping datetime to isoformat method to ensure consistent serialization across all endpoints
- Use Field with default_factory for mutable defaults rather than direct assignment to avoid shared mutable state across model instances
- For score and rating fields, combine Field constraints with enum types to enforce both numeric bounds and categorical validity
- Document field-level descriptions using Field description parameter to populate OpenAPI schema documentation automatically

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the validation library version; locate the lock or resolution artifact to confirm the exact installed version
- Discover the project's test suite location and execute validation-specific test cases that verify Field constraints, type checking, and serialization behavior
- Discover the project's static analysis configuration and run type checking to verify all public API models have complete type annotations

Accept when:
- All public API endpoint models inherit from BaseModel with explicit field types and validation passes for valid inputs while rejecting invalid inputs with clear error messages
- Models with numeric constraints enforce bounds using Field parameters and reject out-of-range values during instantiation
- Datetime fields serialize to ISO format strings in JSON responses without manual conversion code

## Enforcement

- Verified by: Static type checking in continuous integration pipeline verifies all public API models have complete type annotations
- Verified by: Integration tests validate that endpoints reject malformed requests with appropriate HTTP 422 validation error responses
- Verified by: Code review checklist includes verification that new API models follow BaseModel patterns with Field constraints
- Violation handling: Pull requests introducing public API models without BaseModel inheritance are blocked by automated checks
- Violation handling: Runtime validation failures generate structured error responses with field-level detail for API consumers
- Violation handling: Periodic architecture audits identify validation gaps and generate remediation tickets
- Exception process: Request exception through architecture review with performance profiling evidence or migration timeline
- Exception process: Document exception rationale, scope, and expiration date in ADR exceptions registry
- Exception process: Review exceptions quarterly to assess whether conditions for exception still apply