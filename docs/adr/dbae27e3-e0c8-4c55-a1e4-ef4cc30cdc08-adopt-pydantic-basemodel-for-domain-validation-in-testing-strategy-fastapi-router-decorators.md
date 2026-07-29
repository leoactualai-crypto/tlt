# Adopt Pydantic BaseModel for Domain Validation in Testing Strategy: Fastapi Router Decorators

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple FastAPI service endpoints across Discord adapters and MCP services that require structured request/response validation
- Domain models are consistently defined using Pydantic BaseModel with explicit field constraints (Field with ge/le bounds, Optional types, type annotations)
- Integration testing patterns emerge through router endpoint definitions with response_model parameters that enforce contract validation at runtime
- Services handle external inputs from Discord events, HTTP requests, and photo processing workflows requiring input sanitization and type safety
- The pattern appears across 5 files with 92.22% confidence, indicating systematic adoption rather than isolated usage

## Problem Statement

Services processing external inputs from Discord events, HTTP APIs, and asynchronous workflows require a consistent mechanism to validate domain objects, enforce type safety, and provide clear contract definitions for integration testing without manual validation logic scattered across endpoint handlers.

## Decision

1. MUST: FastAPI router decorators MUST specify response_model parameter using the corresponding Pydantic model to enforce contract validation (e.g., @router.post('/', response_model=ReminderResponse))

## Policy Block

- MUST FastAPI router decorators MUST specify response_model parameter using the corresponding Pydantic model to enforce contract validation (e.g., @router.post('/', response_model=ReminderResponse))

In scope:
- All FastAPI router endpoint request/response models
- Domain validation models for Discord adapter services (reminder.py, experience_manager.py, rsvp.py)
- MCP service domain models (photo_processor.py PhotoAnalysisOutput, GenZVibeCheckOutput)
- Service monitoring response models (monitor.py TaskStatusResponse, ServiceStatusResponse)
- Any model representing external API contracts or workflow state

Out of scope:
- Internal data structures not exposed through API boundaries
- Temporary validation logic within function bodies
- Database ORM models (unless also serving as API contracts)
- Configuration objects loaded from environment variables

## Rationale

- Pydantic BaseModel provides automatic validation, serialization, and deserialization with minimal boilerplate, reducing manual validation code across 5 observed service files
- FastAPI's native integration with Pydantic enables automatic OpenAPI schema generation and request/response validation at framework level, improving integration testing reliability
- Field-level constraints (ge, le, description) encode domain rules directly in model definitions, making validation logic explicit and testable independent of endpoint handlers
- The pattern's 92.22% confidence across multiple service types (Discord adapters, MCP services, monitoring) indicates proven effectiveness for heterogeneous input sources

## Consequences

Positive:
- Automatic input validation at API boundaries reduces security vulnerabilities from malformed requests and eliminates manual validation code
- Type safety and IDE autocomplete improve developer experience and reduce runtime type errors
- FastAPI automatic OpenAPI documentation generation provides accurate API contracts for integration testing and client generation
- Centralized domain model definitions serve as single source of truth for data structures across service layers

Negative:
- Pydantic dependency introduces framework coupling and migration cost if validation strategy changes
- Complex validation logic requiring cross-field dependencies or external data may require custom validators, increasing model complexity
- Runtime validation overhead on every request may impact performance for high-throughput endpoints
- Model versioning and backward compatibility require careful management when evolving API contracts

## Alternatives

- Manual validation using isinstance checks and conditional logic within endpoint handlers (rejected)
  Rejected because: Scatters validation logic across multiple files, increases code duplication, lacks automatic serialization, and provides no automatic API documentation generation
  When valid: For simple internal functions with minimal validation requirements not exposed through API boundaries
- Dataclasses with separate validation functions (rejected)
  Rejected because: Requires manual validation implementation, lacks FastAPI integration for automatic request parsing, and does not provide Field-level constraint definitions
  When valid: For internal data structures where serialization and validation are handled separately
- JSON Schema validation with jsonschema library (rejected)
  Rejected because: Separates schema definitions from Python type system, requires manual serialization/deserialization, and lacks IDE type checking support
  When valid: When validating arbitrary JSON documents without corresponding Python models

## Risks

- Pydantic version upgrades may introduce breaking changes in validation behavior or Field API, requiring model updates across multiple services
  Mitigation: Pin Pydantic major version in dependency management, test validation behavior in CI, and review Pydantic changelog before upgrades
  Owner: Engineering team
- Complex nested models with circular dependencies may cause validation errors or infinite recursion
  Mitigation: Use forward references and Pydantic's update_forward_refs() for circular dependencies, limit nesting depth, and test complex models independently
  Owner: Engineering team
- Performance degradation on high-volume endpoints due to validation overhead for large payloads or deeply nested structures
  Mitigation: Profile validation performance in load testing, consider validation caching for repeated patterns, and use Pydantic's Config.validate_assignment=False for internal mutations
  Owner: Engineering team

## Implementation Notes

- Import BaseModel and Field from pydantic, and Optional/List from typing for all domain validation models
- Name request models with 'Create' or 'Update' suffix (e.g., ReminderCreate) and response models with 'Response' suffix (e.g., ReminderResponse) for clarity
- Use Field(ge=X, le=Y) for numeric bounds, Field(description='...') for API documentation, and Optional[T] = None for optional fields with defaults
- Apply response_model parameter to all FastAPI router decorators (@router.post, @router.get) to enforce output validation and enable automatic schema generation
- Test models independently by instantiating with valid and invalid data to verify Field constraints trigger ValidationError as expected

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Create|Response|Output|Update)' | wc -l
- grep -r '@router\.(post|get|put|delete)' monorepo/tlt --include='*.py' | grep 'response_model=' | wc -l
- grep -r 'Field(.*ge=.*le=' monorepo/tlt --include='*.py' | wc -l

Accept when:
- All FastAPI router endpoints define request/response models as Pydantic BaseModel subclasses with response_model parameter specified
- Numeric fields with bounded ranges use Field(ge=X, le=Y) constraints consistently across domain models
- Grep commands return non-zero counts indicating presence of BaseModel inheritance, response_model usage, and Field constraints

## Enforcement

- Verified by: Code review checklist requiring Pydantic BaseModel for new API endpoints
- Verified by: CI pipeline grep checks verifying response_model presence on router decorators
- Verified by: Static analysis with mypy to enforce type annotations on model fields
- Violation handling: CI build fails if new router endpoints lack response_model parameter
- Violation handling: Code review blocks merge if domain models lack Field constraints for bounded numeric values
- Violation handling: Linting warnings for BaseModel subclasses without type annotations
- Exception process: Document exception rationale in code comments for endpoints with manual validation
- Exception process: Obtain architecture review approval for alternative validation approaches
- Exception process: Add exception to .pylintrc or mypy.ini with justification comment