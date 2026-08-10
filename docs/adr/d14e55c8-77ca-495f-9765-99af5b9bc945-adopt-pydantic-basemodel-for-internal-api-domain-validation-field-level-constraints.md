# Adopt Pydantic BaseModel for Internal API Domain Validation: Field Level Constraints

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all internal API implementations requiring domain validation.

## Context

- Internal APIs across agent nodes, service endpoints, and MCP services require structured data validation to ensure type safety and contract enforcement at runtime boundaries
- Multiple service layers (ambient event agent reasoning, photo processing workflows, Discord adapters, monitoring endpoints) independently adopted schema-based validation for request/response models
- Domain models require field-level constraints (numeric ranges, required/optional fields, nested structures) that cannot be expressed through type hints alone
- API endpoints expose structured responses to external clients and internal consumers, necessitating consistent serialization and validation behavior
- Agent decision-making workflows depend on validated structured outputs to coordinate actions across message sending, timer scheduling, and tool execution

## Problem Statement

Internal APIs must validate complex domain models with field-level constraints, nested structures, and conditional requirements while maintaining type safety across service boundaries. Without declarative schema validation, services must implement manual validation logic that is error-prone, inconsistent, and difficult to maintain across multiple endpoints and workflows.

## Decision

1. MUST: Field-level constraints MUST be declared using the validation framework's field descriptor mechanism with explicit constraint parameters for numeric ranges, string patterns, and collection sizes

## Policy Block

- MUST Field-level constraints MUST be declared using the validation framework's field descriptor mechanism with explicit constraint parameters for numeric ranges, string patterns, and collection sizes

In scope:
- All FastAPI endpoint request and response models
- Agent reasoning decision structures and workflow state models
- MCP service input/output schemas
- CloudEvent context and domain event models
- Health check and monitoring response structures

Out of scope:
- External API client models defined by third-party services
- Database ORM models with framework-specific validation
- Configuration file schemas validated at load time
- Internal data transfer objects used exclusively within single-module boundaries

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily use dictionary-based validation
- EXC-002: Performance-critical hot paths with profiled validation overhead may defer validation to boundary layers

## Rationale

- Evidence shows 8 files across agent nodes, service endpoints, and adapters consistently using BaseModel inheritance with Field descriptors for constraint declaration, demonstrating established pattern adoption
- Declarative field constraints (ge=0.0, le=1.0 for confidence scores, required/optional distinctions) eliminate manual validation code and provide self-documenting schemas
- Type-safe model composition enables compile-time checking and IDE support while maintaining runtime validation guarantees at service boundaries
- Structured validation errors provide actionable feedback to API consumers and enable consistent error handling across internal service integrations

## Consequences

Positive:
- Type safety and runtime validation are enforced consistently across all internal API boundaries without manual validation code
- API contracts are self-documenting through field descriptions and constraint declarations visible in model definitions
- Validation errors provide structured, actionable feedback with field-level error messages for debugging and client integration
- Model composition through typed references enables reusable domain structures and reduces duplication across services

Negative:
- Validation framework dependency becomes a hard requirement for all internal API implementations, increasing coupling
- Complex validation logic may require custom validators that bypass declarative constraint benefits
- Runtime validation overhead adds latency to request processing, particularly for deeply nested structures
- Schema evolution requires careful management of optional fields and default values to maintain backward compatibility

## Alternatives

- Use dataclasses with manual validation functions for each model (rejected)
  Rejected because: Manual validation code is error-prone, inconsistent across services, and lacks declarative constraint expression. Evidence shows no adoption of this pattern.
  When valid: May be appropriate for internal data structures with no external API exposure
- Rely solely on type hints without runtime validation (rejected)
  Rejected because: Type hints provide static checking but no runtime guarantees. Internal APIs require validation at service boundaries to handle untrusted or malformed data.
  When valid: Acceptable for tightly-coupled internal modules with guaranteed type safety
- Use JSON Schema with separate validation layer (rejected)
  Rejected because: Separates schema definition from Python type system, requiring duplicate maintenance. Evidence shows preference for integrated validation within model classes.
  When valid: May be appropriate when schema must be shared with non-Python services or generated from OpenAPI specifications

## Risks

- Validation framework version incompatibilities may break existing models during dependency upgrades
  Mitigation: Pin validation framework version in dependency manifest. Test all API models against new versions before upgrading. Maintain version-specific migration guides.
  Owner: Engineering team
- Performance degradation in high-throughput endpoints due to validation overhead on large nested structures
  Mitigation: Profile validation performance in critical paths. Consider boundary validation strategy where validation occurs once at entry points. Cache validated models where appropriate.
  Owner: Service owners
- Complex cross-field validation logic may become difficult to maintain within model classes
  Mitigation: Extract complex validation to dedicated validator functions. Document validation logic clearly. Consider separating validation concerns from domain models for highly complex cases.
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
- Define all API models in dedicated schema modules separate from business logic to enable reuse across service layers and maintain clear separation of concerns
- Use field descriptions consistently to document expected values, constraints, and business rules. These descriptions serve as inline API documentation for consumers.
- For agent decision models with conditional field requirements, document which fields are required for each decision type in the model docstring to guide consumers
- When composing models with nested structures, prefer typed model references over generic dictionaries to maintain validation guarantees throughout the object graph

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock artifact. Resolve the validation framework's exact installed version. Locate the project's test suite discovery mechanism and execute validation-related tests.
- Discover the project's static type checking configuration. Execute the type checker against all modules containing API model definitions to verify type safety.
- Discover the project's code search or analysis tooling. Search for all classes inheriting from the validation base model class. Verify each includes field constraint declarations and type annotations.

Accept when:
- All internal API request and response models inherit from the validation framework base class with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs

## Enforcement

- Verified by: Code review checklist requires validation model definitions for all new API endpoints
- Verified by: Static type checking in continuous integration pipeline verifies model type annotations
- Verified by: Integration tests validate that API endpoints enforce model constraints and return structured validation errors
- Violation handling: Pull requests introducing API endpoints without validated models are blocked until models are added
- Violation handling: Type checking failures in CI pipeline prevent merge until resolved
- Violation handling: Runtime validation errors in production are logged with structured error details for debugging and monitoring
- Exception process: Service owner documents exception rationale and alternative validation strategy in service README
- Exception process: Architecture review approves exception for performance-critical paths with profiling data
- Exception process: Exception approval includes timeline for migration to standard validation approach where applicable