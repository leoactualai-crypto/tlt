# Standardize Pydantic BaseModel for Input Validation in Logged Service Endpoints: Response Models Mirror

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all FastAPI service endpoints that implement logging via logging.getLogger(__name__) or loguru.logger and accept external input.

## Context

- Five service modules (reminder.py, experience_manager.py, photo_processor.py, rsvp.py, monitor.py) implement FastAPI routers with structured logging using either logging.getLogger(__name__) or loguru.logger
- Each module defines Pydantic BaseModel subclasses for request/response validation (ReminderCreate, ExperienceCreate, PhotoAnalysisOutput, ReactionUpdate, TaskStatusResponse) with explicit field constraints
- All modules handle external input from Discord adapters or HTTP clients and log operations at info/error levels, creating a correlation between input validation and observability
- The codebase uses Field validators with constraints (ge=0.0, le=1.0) and type annotations (int, str, datetime, Optional[List[str]]) to enforce input contracts before processing
- Error handling patterns consistently raise HTTPException with status codes and detail messages, which are logged alongside validation failures

## Problem Statement

Service endpoints that accept external input and implement structured logging need a consistent mechanism to validate input contracts before processing, preventing invalid data from propagating through logged operations and ensuring that log entries reference validated, type-safe data structures rather than arbitrary dictionaries or unvalidated payloads.

## Decision

1. SHOULD: Response models SHOULD mirror request models with additional fields (id, status, created_at, updated_at) to maintain validation consistency across request/response cycles

## Policy Block

- SHOULD Response models SHOULD mirror request models with additional fields (id, status, created_at, updated_at) to maintain validation consistency across request/response cycles

In scope:
- FastAPI router endpoints in Discord adapters (reminder.py, experience_manager.py, rsvp.py)
- MCP service endpoints with external HTTP clients (photo_processor.py)
- Monitoring and health check endpoints that return structured status (monitor.py)
- Any endpoint that logs request/response data using logging or loguru

Out of scope:
- Internal function calls that do not cross service boundaries
- Logging statements that do not reference external input
- Configuration loading or environment variable parsing
- Database query results or internal data transformations

Exceptions:
- EXC-001: Endpoint accepts raw binary data (images, files) that cannot be represented as Pydantic models
- EXC-002: Prototype or experimental endpoints explicitly marked as unstable

## Rationale

- The pattern appears consistently across 5 files with 92.22% confidence, indicating an established practice rather than isolated implementation
- Pydantic validation occurs before logging operations, ensuring that logged data references validated structures and reducing noise from malformed input in log streams
- Field-level constraints (ge=0.0, le=1.0, type annotations) prevent invalid data from reaching business logic, making log entries more reliable for debugging and monitoring
- The correlation between BaseModel validation and HTTPException handling creates a consistent error boundary that is observable through structured logs

## Consequences

Positive:
- Log entries reference type-safe, validated data structures, improving log reliability and reducing debugging time
- Input validation failures are caught early and logged consistently, preventing invalid data from propagating through service layers
- Pydantic models serve as self-documenting contracts for both API consumers and log analysis tools
- Field constraints and type annotations enable automatic OpenAPI schema generation, aligning validation, documentation, and logging

Negative:
- Adds boilerplate for defining BaseModel classes, increasing initial development time for new endpoints
- Pydantic validation overhead may impact latency for high-throughput endpoints with complex nested models
- Validation errors generate log entries that may increase log volume in scenarios with frequent malformed input
- Tight coupling between validation models and logging makes it harder to change validation logic without affecting log schema

## Alternatives

- Use FastAPI dependency injection with custom validator functions instead of Pydantic models (rejected)
  Rejected because: Custom validators lack automatic OpenAPI schema generation and require manual logging integration, reducing consistency across endpoints
  When valid: Valid for endpoints with complex validation logic that cannot be expressed in Pydantic Field constraints
- Validate input using raw dictionaries with manual type checks and log validation failures separately (rejected)
  Rejected because: Manual validation is error-prone, lacks type safety, and creates inconsistent logging patterns across services
  When valid: Not recommended; only valid for legacy endpoints being migrated to Pydantic
- Use dataclasses with separate validation decorators instead of Pydantic BaseModel (rejected)
  Rejected because: Dataclasses lack built-in validation, JSON serialization, and FastAPI integration, requiring additional libraries and reducing cohesion
  When valid: Valid for internal data structures that do not cross service boundaries or require logging

## Risks

- Pydantic validation performance overhead may become a bottleneck for high-throughput endpoints with complex nested models
  Mitigation: Profile endpoint latency and consider caching validated models or using Pydantic v2 with Rust-based validation for performance-critical paths
  Owner: Backend engineering team
- Changes to validation models may break log parsing tools or monitoring dashboards that depend on specific field names or types
  Mitigation: Version validation models explicitly and maintain backward compatibility for logged fields; use schema evolution strategies for breaking changes
  Owner: Platform engineering team
- Overly strict validation constraints may reject valid edge-case input, generating false-positive error logs
  Mitigation: Review Field constraints during code review; implement monitoring for validation failure rates and adjust constraints based on production data
  Owner: Service owners

## Implementation Notes

- Define BaseModel subclasses in the same module as the router to maintain locality between validation and endpoint logic
- Use Field(description=...) for all fields to generate self-documenting OpenAPI schemas and provide context in validation error logs
- Log validation failures at error level with logger.error(f'Validation failed: {e}') to distinguish from business logic errors
- For nested models (e.g., Optional[List[str]]), define separate BaseModel classes rather than inline type annotations to improve validation error messages
- Include response_model=... in @router decorators to enable automatic response validation and consistent logging of output data

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt/adapters/ monorepo/tlt/services/ | grep -v '__pycache__' | wc -l
- grep -r '@router\.(get|post|put|delete)' monorepo/tlt/ | grep 'response_model=' | wc -l
- grep -r 'Field(' monorepo/tlt/ | grep -E '(ge=|le=|description=)' | wc -l
- grep -r 'logging\.getLogger\|from loguru import logger' monorepo/tlt/ | wc -l

Accept when:
- All FastAPI router endpoints that implement logging have corresponding Pydantic BaseModel definitions for request/response payloads
- At least 80% of BaseModel fields include Field constraints or description metadata
- Validation failures are logged at error level with sufficient context to correlate with input payloads
- Response models are specified in @router decorators using response_model parameter

## Enforcement

- Verified by: Code review checklist requiring Pydantic BaseModel for all new FastAPI endpoints with logging
- Verified by: CI pipeline grep checks for @router decorators without response_model parameters
- Verified by: Static analysis via mypy to enforce type annotations on all BaseModel fields
- Verified by: Periodic audit of log entries to verify structured data references validated models
- Violation handling: CI pipeline fails if new router endpoints lack BaseModel definitions
- Violation handling: Code review blocks merge if validation models are missing Field constraints for numeric fields
- Violation handling: Runtime validation errors are logged and monitored; repeated failures trigger alerts for service owners
- Violation handling: Quarterly review of validation failure rates to identify endpoints requiring constraint adjustments
- Exception process: Submit exception request to tech lead with documented rationale and alternative validation mechanism
- Exception process: Exceptions must include compensating controls (e.g., manual validation, enhanced logging) and expiration date
- Exception process: Approved exceptions are recorded in service documentation and reviewed quarterly for removal