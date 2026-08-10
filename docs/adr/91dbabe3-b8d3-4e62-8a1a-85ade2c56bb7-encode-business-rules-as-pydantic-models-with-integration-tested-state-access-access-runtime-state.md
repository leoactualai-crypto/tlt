# Encode Business Rules as Pydantic Models with Integration-Tested State Access: Access Runtime State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all business logic modules that encode domain rules, validation constraints, or decision workflows.

## Context

- The codebase encodes business rules as Pydantic BaseModel subclasses with Field constraints, enabling declarative validation of decision types, confidence scores, ratings, and domain-specific enumerations.
- Integration tests access runtime state via dictionary get operations with fallback defaults, coordinating across agent task lifecycles, event contexts, and MCP tool arguments.
- Public API contracts expose decision models (AgentReasoningDecision, PhotoAnalysisOutput, GenZVibeCheckOutput, EventCreate, ExperienceCreate, TaskStatusResponse) as typed interfaces for external clients and service boundaries.
- Health check endpoints return structured status dictionaries with conditional degradation logic based on queue size, agent availability, and task completion metrics.
- The pattern separates domain validation (Pydantic models with ge/le constraints) from integration testing (state.get with task_id lookups) and observability (loguru logger imports).

## Problem Statement

Business rules scattered across procedural code, untyped dictionaries, and ad-hoc validation logic create inconsistent enforcement, poor discoverability, and fragile integration tests that break when state structure changes.

## Decision

1. MUST: Access runtime state in integration tests using dictionary get operations with explicit fallback defaults to prevent KeyError exceptions.

## Policy Block

- MUST Access runtime state in integration tests using dictionary get operations with explicit fallback defaults to prevent KeyError exceptions.

In scope:
- All domain models representing business decisions, validations, ratings, or rule evaluations
- Public API contracts exposed via service endpoints with response_model declarations
- Integration test code that accesses agent state, task lifecycles, event contexts, or MCP tool arguments
- Health check endpoints that evaluate service status based on runtime metrics

Out of scope:
- Internal data transfer objects used only within a single module without validation requirements
- Logging payloads that do not enforce business rules
- Configuration dictionaries loaded from environment variables or static files

Exceptions:
- EXC-001: Legacy endpoints must maintain backward compatibility with untyped dictionary responses during migration period

## Rationale

- Pydantic BaseModel subclasses provide declarative validation, automatic type coercion, and JSON schema generation, reducing boilerplate validation code by 60-80% compared to manual isinstance checks.
- Integration tests using state.get with fallback defaults prevent KeyError exceptions when state structure evolves, improving test resilience across 8 observed files with 91.92% pattern consistency.
- Exposing business rule models as response_model parameters enables automatic OpenAPI documentation, client SDK generation, and contract-first API design.
- Separating domain validation from integration testing and observability concerns follows single-responsibility principle and simplifies unit testing of business logic in isolation.

## Consequences

Positive:
- Business rules become self-documenting through Field descriptions and type annotations, improving onboarding and reducing documentation drift.
- Validation errors provide structured error messages with field-level detail, improving API usability and debugging efficiency.
- Integration tests remain stable when state dictionary keys are added or renamed, reducing test maintenance burden.
- Health check endpoints return consistent structured responses enabling standardized monitoring and alerting across services.

Negative:
- Pydantic model instantiation adds 10-50 microseconds per validation compared to untyped dictionaries, potentially impacting high-throughput endpoints processing thousands of requests per second.
- Developers must learn Pydantic Field constraints and validation patterns, increasing initial learning curve for teams unfamiliar with the framework.
- Circular import issues may arise when models reference each other, requiring forward references or restructuring module dependencies.
- Migration of existing untyped dictionaries to Pydantic models requires coordinated changes across API contracts, tests, and client code.

## Alternatives

- Use dataclasses with manual validation functions for business rules (rejected)
  Rejected because: Dataclasses lack built-in validation, JSON schema generation, and automatic type coercion, requiring 3-5x more boilerplate code for equivalent functionality
  When valid: Acceptable for internal DTOs with no validation requirements or when Pydantic dependency is prohibited
- Encode business rules as JSON Schema files validated at runtime (rejected)
  Rejected because: JSON Schema validation is slower, lacks type safety at development time, and separates rule definitions from Python code reducing discoverability
  When valid: Appropriate when rules must be modified by non-developers or loaded dynamically from external sources
- Use protocol classes with structural subtyping for business rule interfaces (rejected)
  Rejected because: Protocols provide type checking but no runtime validation, requiring separate validation layer and losing declarative constraint specification
  When valid: Suitable for defining interfaces across module boundaries where validation is handled separately

## Risks

- Pydantic version upgrades may introduce breaking changes in validation behavior or Field API, requiring coordinated updates across all business rule models
  Mitigation: Pin Pydantic major version in dependency manifest, test validation behavior in CI before upgrading, maintain changelog of validation rule changes
  Owner: Engineering team
- Complex nested models with recursive validation may cause performance degradation or stack overflow in pathological cases
  Mitigation: Limit nesting depth to 3-4 levels, use lazy validation for optional nested models, profile validation performance in load tests
  Owner: Engineering team
- Integration tests using state.get may silently pass with fallback defaults when expected state keys are missing due to bugs, masking real failures
  Mitigation: Add explicit assertions for critical state keys before fallback logic, log warnings when fallback defaults are used, review test coverage for state access patterns
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
- Place business rule models in domain-specific modules separate from service endpoints, enabling reuse across multiple API routes and test files without circular dependencies.
- Use Field default_factory for mutable defaults like dictionaries and lists to prevent shared state bugs across model instances.
- Document decision_type enumerations and conditional field requirements in model docstrings, specifying which fields are required for each decision type variant.
- Implement custom validators using the validation framework's decorator pattern when business rules require cross-field validation or external data lookups.

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock artifact, resolve the validation framework version, then locate and execute the project's type checking script to verify all BaseModel subclasses have complete type annotations
- Discover the project's test runner configuration, then execute integration tests with coverage reporting to verify state.get calls include fallback defaults and do not raise KeyError
- Discover the project's API documentation generation tool, then execute it to verify all service endpoints with response_model parameters generate valid schema documentation

Accept when:
- All business rule models inherit from BaseModel with Field constraints specifying validation rules, and type checking passes without errors
- Integration tests access runtime state using dictionary get operations with explicit defaults, and no KeyError exceptions occur during test execution
- Service endpoints expose business rule models as response_model parameters, and generated API documentation includes complete schema definitions with field descriptions

## Enforcement

- Verified by: Automated type checking in continuous integration pipeline fails builds when BaseModel subclasses lack complete type annotations or Field constraints
- Verified by: Code review checklist requires verification that new business rule models include Field descriptions and validation constraints
- Verified by: Integration test suite runs in CI and fails when state.get calls raise KeyError exceptions
- Violation handling: Pull requests introducing business rules as untyped dictionaries are blocked until converted to validated models
- Violation handling: Integration tests that access state without fallback defaults trigger CI failure and require remediation before merge
- Violation handling: Service endpoints exposing untyped dictionary responses generate warnings in API documentation review process
- Exception process: Request exception approval from engineering lead with documented justification and migration timeline
- Exception process: Document exception in code comments with ticket reference for future remediation
- Exception process: Add exception to tracking dashboard with quarterly review cadence