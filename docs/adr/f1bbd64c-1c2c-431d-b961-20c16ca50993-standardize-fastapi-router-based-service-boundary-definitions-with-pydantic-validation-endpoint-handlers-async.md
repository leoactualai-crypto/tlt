# Standardize FastAPI Router-Based Service Boundary Definitions with Pydantic Validation: Endpoint Handlers Async

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all service boundary implementations in the monorepo.

## Context

- The monorepo contains multiple service adapters (discord_adapter) and services (tlt_service) that expose HTTP endpoints for external integration and monitoring
- Services require consistent input validation to prevent injection attacks, type confusion, and malformed data from reaching business logic
- FastAPI with Pydantic BaseModel provides declarative schema validation at service boundaries, reducing manual validation code and security vulnerabilities
- Four files demonstrate consistent use of FastAPI routers with typed request/response models (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) and HTTPException error handling
- Service definitions use async endpoint handlers with explicit response_model declarations, enabling OpenAPI documentation and runtime validation

## Problem Statement

Services exposed via HTTP endpoints require robust input validation and type safety at their boundaries to prevent security vulnerabilities, data corruption, and runtime errors. Without standardized validation patterns, each service implementation may handle validation inconsistently, leading to gaps in security coverage and increased maintenance burden.

## Decision

1. SHOULD: Endpoint handlers SHOULD be async functions to support concurrent request processing

## Policy Block

- SHOULD Endpoint handlers SHOULD be async functions to support concurrent request processing

In scope:
- All HTTP endpoints in adapters (discord_adapter) and services (tlt_service)
- Public API contracts exposed for external integration
- Health check and monitoring endpoints
- CRUD operations on domain entities (reminders, experiences, reactions, tasks)

Out of scope:
- Internal function calls within a service that do not cross HTTP boundaries
- Discord bot event handlers that receive Discord.py objects directly
- Background tasks and scheduled jobs that do not accept HTTP input
- Database model definitions (separate from API validation models)

Exceptions:
- EXC-001: Legacy endpoints require gradual migration to Pydantic validation
- EXC-002: Proxy endpoints that forward raw requests to external services without inspection

## Rationale

- Evidence shows 4 files consistently using FastAPI routers with Pydantic BaseModel validation (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse), indicating an established pattern with 92.22% confidence
- Pydantic validation at service boundaries prevents type confusion, injection attacks, and malformed data from reaching business logic, reducing security vulnerabilities
- FastAPI's automatic OpenAPI schema generation from Pydantic models provides self-documenting APIs and enables client code generation
- HTTPException usage with specific status codes (404, 500, 503) provides consistent error handling and appropriate HTTP semantics for monitoring and debugging

## Consequences

Positive:
- Input validation is enforced declaratively at service boundaries, reducing manual validation code and security vulnerabilities
- Type safety is guaranteed for all HTTP requests and responses, preventing runtime type errors
- OpenAPI documentation is automatically generated from Pydantic models, improving API discoverability and client integration
- Consistent error handling with HTTPException provides predictable behavior for API consumers and monitoring systems

Negative:
- Pydantic model definitions add boilerplate code for each endpoint, increasing initial development time
- Complex validation logic may require custom validators, adding complexity to model definitions
- Tight coupling to FastAPI and Pydantic makes migration to alternative frameworks more difficult
- Response model validation adds runtime overhead for serialization, though typically negligible for most use cases

## Alternatives

- Use raw request.json() parsing with manual validation in endpoint handlers (rejected)
  Rejected because: Manual validation is error-prone, inconsistent across endpoints, and provides no automatic documentation or type safety guarantees
  When valid: Never recommended for new code; only acceptable in legacy endpoints undergoing migration
- Use dataclasses with manual validation instead of Pydantic BaseModel (rejected)
  Rejected because: Dataclasses lack runtime validation, automatic coercion, and FastAPI integration for response_model serialization
  When valid: Only for internal data structures that do not cross HTTP boundaries
- Use JSON Schema validation with jsonschema library (rejected)
  Rejected because: JSON Schema validation is separate from type annotations, provides no IDE support, and lacks FastAPI integration for automatic documentation
  When valid: Only when validating external JSON documents that are not part of the API contract

## Risks

- Developers may bypass Pydantic validation by accessing raw request bodies or using untyped parameters
  Mitigation: Enforce validation through code review, linting rules that detect raw request access, and security testing that attempts to send malformed payloads
  Owner: Engineering team and security reviewers
- Complex nested validation models may become difficult to maintain and understand
  Mitigation: Establish conventions for model organization, use composition over deep nesting, and document complex validation logic with examples
  Owner: Engineering team and API design reviewers
- Pydantic version upgrades may introduce breaking changes in validation behavior
  Mitigation: Pin Pydantic major version in dependencies, test validation behavior in CI, and review Pydantic changelog before upgrades
  Owner: Engineering team and dependency management process

## Implementation Notes

- Create Pydantic models in a models.py or schemas.py module adjacent to router definitions for discoverability
- Use descriptive field names and include Field() with description parameter for automatic API documentation
- For endpoints that modify state, use separate Create/Update models rather than reusing response models to enforce immutability
- Include example values in Pydantic models using Config.schema_extra for better OpenAPI documentation and testing

## Continuation Context


Verify commands:
- grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' | grep -v 'response_model=' && echo 'Found endpoints without response_model' || echo 'All endpoints have response_model'
- grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | wc -l
- grep -r 'raise HTTPException' --include='*.py' monorepo/tlt/ | wc -l

Accept when:
- All HTTP endpoints use FastAPI router decorators with explicit response_model declarations
- All request bodies are validated using Pydantic BaseModel subclasses with type annotations
- Error conditions raise HTTPException with appropriate status codes rather than returning error dictionaries

## Enforcement

- Verified by: Code review checklist requiring Pydantic validation for all new endpoints
- Verified by: CI pipeline grep checks for endpoints without response_model declarations
- Verified by: Security testing that attempts to send malformed payloads to all endpoints
- Verified by: Static analysis with mypy to verify type annotations on endpoint handlers
- Violation handling: CI pipeline fails if endpoints lack response_model declarations
- Violation handling: Code review blocks merge if Pydantic validation is missing for request bodies
- Violation handling: Security testing failures trigger immediate remediation for endpoints accepting unvalidated input
- Violation handling: Quarterly audit identifies and tracks remediation of legacy endpoints without validation
- Exception process: Developer submits exception request with security justification to tech lead
- Exception process: Security team reviews exception for injection and data corruption risks
- Exception process: Approved exceptions are documented in endpoint docstring with expiration date
- Exception process: Exception registry is reviewed quarterly to ensure timely remediation