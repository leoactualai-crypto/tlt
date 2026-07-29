# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Validation Models Include

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all API boundary implementations and domain model definitions within the codebase.

## Context

- The codebase implements multiple FastAPI-based service adapters (discord_adapter, tlt_service) that expose HTTP endpoints requiring structured request/response validation
- Domain models require runtime type checking, field constraints (ge, le, Optional), and automatic serialization for API contracts across 5 detected files
- Services handle external client interactions through Discord bot integrations, MCP services, and monitoring endpoints requiring consistent validation patterns
- The system processes user-generated content (reminders, experiences, reactions, photos) where input validation prevents malformed data from entering business logic
- Async/await concurrency patterns are pervasive across all detected files, requiring validation frameworks compatible with asyncio-based request handling

## Problem Statement

API boundaries in FastAPI services require a standardized approach to validate incoming requests, enforce domain constraints, serialize responses, and provide clear contract definitions. Without consistent validation, services risk accepting malformed data, producing inconsistent error messages, and creating maintenance overhead through manual validation code scattered across endpoints.

## Decision

1. SHOULD: Validation models SHOULD include descriptive field names and Field descriptions to generate self-documenting API contracts

## Policy Block

- SHOULD Validation models SHOULD include descriptive field names and Field descriptions to generate self-documenting API contracts

In scope:
- All FastAPI router endpoint request/response models
- Domain entities exposed through HTTP APIs (ReminderCreate, ExperienceResponse, TaskStatusResponse, etc.)
- Data transfer objects crossing service boundaries
- Configuration models requiring validation (PhotoAnalysisOutput, GenZVibeCheckOutput)
- Monitoring and health check response structures

Out of scope:
- Internal business logic classes not exposed through APIs
- Database ORM models (unless also serving as API models)
- Pure data structures used only within single modules
- Third-party library models (discord.py models, etc.)

Exceptions:
- EXC-001: Legacy endpoints require gradual migration from dict-based validation
- EXC-002: Performance-critical paths demonstrate measurable Pydantic overhead (>10ms p99 latency)

## Rationale

- Evidence shows consistent pattern across 5 files (92.22% confidence) using Pydantic BaseModel for domain validation with Field constraints, indicating established architectural practice
- FastAPI's native integration with Pydantic provides automatic request validation, response serialization, and OpenAPI schema generation without additional code
- Detected models (ReminderCreate, ExperienceResponse, PhotoAnalysisOutput, TaskStatusResponse) demonstrate Field-level constraints (ge=0.0, le=1.0) enforcing domain invariants at API boundaries
- The pattern separates validation concerns from business logic, enabling reusable models across multiple endpoints (router.post, router.get) as evidenced in all detected files

## Consequences

Positive:
- Automatic runtime validation catches malformed requests before reaching business logic, reducing defensive programming overhead
- Self-documenting API contracts through Field descriptions and type hints improve developer experience and reduce documentation drift
- FastAPI generates accurate OpenAPI schemas automatically, enabling client code generation and interactive API documentation
- Consistent error responses across all endpoints through Pydantic's standardized validation error format

Negative:
- Pydantic validation adds runtime overhead (typically 1-5ms per request) compared to unvalidated dictionaries
- Complex nested models can create verbose class definitions requiring more boilerplate than simple dictionaries
- Pydantic version upgrades may introduce breaking changes in validation behavior requiring careful testing
- Developers must learn Pydantic-specific patterns (Field, validators, Config) beyond standard Python type hints

## Alternatives

- Use dataclasses with manual validation logic in endpoint handlers (rejected)
  Rejected because: Requires duplicated validation code across endpoints, no automatic OpenAPI schema generation, and lacks Field-level constraint enforcement
  When valid: Simple internal services with no external API documentation requirements
- Use marshmallow schemas for validation and serialization (rejected)
  Rejected because: FastAPI has first-class Pydantic integration; marshmallow requires additional adapter code and lacks native async support
  When valid: Projects already heavily invested in marshmallow ecosystem or requiring marshmallow-specific features
- Use TypedDict with runtime type checkers like typeguard (rejected)
  Rejected because: TypedDict provides static type hints only; runtime validation requires separate integration and lacks Field constraint support
  When valid: Type-checking-only scenarios where runtime validation is handled elsewhere

## Risks

- Pydantic v2 migration introduces breaking changes in validation behavior and performance characteristics
  Mitigation: Pin Pydantic version in requirements.txt, establish comprehensive validation test suite before upgrading, review migration guide
  Owner: Engineering team
- Complex nested models with circular references cause validation errors or infinite recursion
  Mitigation: Use Pydantic's update_forward_refs() for circular dependencies, prefer flat models at API boundaries, document nesting limits
  Owner: API design reviewers
- Validation overhead becomes bottleneck in high-throughput endpoints (>1000 req/s)
  Mitigation: Profile validation performance in load tests, consider model_validate() vs model_validate_json() optimization, document exception process for performance-critical paths
  Owner: Performance engineering team

## Implementation Notes

- Import BaseModel from pydantic and define request/response classes inheriting from it before router endpoint definitions
- Use Field() for constraints: Field(ge=0.0, le=1.0, description='...') for numeric bounds, Field(default=None) for optional fields
- Declare response_model parameter in FastAPI decorators: @router.post('/', response_model=YourModel) for automatic validation
- For datetime fields, import datetime from standard library; Pydantic handles ISO 8601 string parsing automatically
- Test validation by sending malformed requests and verifying FastAPI returns 422 Unprocessable Entity with detailed error messages

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l
- grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l
- python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'

Accept when:
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify response_model parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules

## Enforcement

- Verified by: Code review checklist requiring Pydantic models for all new API endpoints
- Verified by: Automated linting rule detecting FastAPI routes without response_model parameter
- Verified by: CI pipeline grep checks counting BaseModel usage in modified API files
- Violation handling: PR comments requesting Pydantic model addition before merge approval
- Violation handling: CI failure on detection of unvalidated dictionary parameters in FastAPI routes
- Violation handling: Architecture review escalation for repeated violations or exception requests
- Exception process: Submit exception request to tech lead with performance benchmark data or migration timeline
- Exception process: Document exception in code with EXC-ID reference and justification comment
- Exception process: Review exceptions quarterly to assess migration progress or performance improvements