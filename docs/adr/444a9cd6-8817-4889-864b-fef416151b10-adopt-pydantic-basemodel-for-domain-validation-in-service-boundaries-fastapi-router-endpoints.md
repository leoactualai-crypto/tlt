# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Fastapi Router Endpoints

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all service boundary implementations and domain validation logic.

## Context

- The codebase contains multiple FastAPI service adapters (discord_adapter/reminder.py, discord_adapter/rsvp.py, discord_adapter/event.py) and service endpoints (tlt_service/event_manager.py, mcp_services/photo_vibe_check/photo_processor.py) that require structured input validation and response serialization
- All detected files use Pydantic BaseModel classes to define domain entities with explicit field types, constraints, and validation rules (ReminderCreate, ReminderResponse, ReactionUpdate, EventCreate, EventResponse, PhotoAnalysisOutput, GenZVibeCheckOutput, TaskResponse)
- FastAPI router endpoints consistently declare response_model parameters using these Pydantic models, establishing contracts between HTTP boundaries and domain logic
- The pattern appears across 5 files with 92% confidence, indicating systematic adoption rather than isolated usage
- Domain validation classes use Pydantic Field constraints (ge, le, description) to enforce business rules at the schema level, separating validation logic from business logic

## Problem Statement

Service boundaries require consistent mechanisms for validating incoming data, serializing outgoing responses, and documenting API contracts. Without a standardized approach, validation logic becomes scattered across handler functions, error handling becomes inconsistent, and API contracts lack machine-readable documentation. The system needs a declarative validation paradigm that integrates with FastAPI's dependency injection and automatic OpenAPI generation while maintaining type safety.

## Decision

1. MUST: FastAPI router endpoints MUST declare response_model parameters using Pydantic models to establish explicit API contracts

## Policy Block

- MUST FastAPI router endpoints MUST declare response_model parameters using Pydantic models to establish explicit API contracts

In scope:
- All FastAPI router endpoints accepting request bodies
- All FastAPI router endpoints returning structured responses
- Domain entities used in service-to-service communication
- Data transfer objects crossing HTTP boundaries
- Configuration objects requiring validation

Out of scope:
- Internal function parameters within a single module
- Database ORM models (SQLAlchemy, Django ORM)
- Simple data structures with no validation requirements
- Logging and debugging data structures

Exceptions:
- EXC-001: Legacy endpoints require gradual migration from untyped dictionaries to Pydantic models
- EXC-002: Performance-critical paths require raw dictionary access to avoid serialization overhead

## Rationale

- Evidence shows consistent adoption across 5 files spanning Discord adapters, event management, and photo processing services, indicating this pattern successfully addresses validation needs across diverse service types
- Pydantic BaseModel integration with FastAPI provides automatic request validation, response serialization, and OpenAPI schema generation, reducing boilerplate and improving API documentation quality
- Field-level constraints (ge=0.0, le=1.0 for scores, explicit type annotations) move validation logic from imperative code to declarative schemas, improving maintainability and reducing validation bugs
- Separation of Create/Response models (ReminderCreate vs ReminderResponse, EventCreate vs EventResponse) establishes clear boundaries between mutable input and immutable output, preventing accidental field exposure

## Consequences

Positive:
- Automatic validation at service boundaries reduces manual validation code and catches type errors before business logic execution
- FastAPI automatically generates OpenAPI schemas from Pydantic models, ensuring API documentation stays synchronized with implementation
- Type annotations enable static analysis tools (mypy, pyright) to catch type errors at development time
- Declarative Field constraints make business rules explicit and self-documenting in the schema definition

Negative:
- Pydantic validation adds serialization overhead compared to raw dictionary access, potentially impacting high-throughput endpoints
- Complex validation logic requiring cross-field dependencies may require custom validators, reducing the declarative simplicity
- Tight coupling to Pydantic means migration to alternative validation frameworks requires significant refactoring
- Nested model validation can produce verbose error messages that require additional processing for user-friendly error responses

## Alternatives

- Use dataclasses with manual validation in endpoint handlers (rejected)
  Rejected because: Requires duplicating validation logic across handlers, lacks automatic OpenAPI generation, and provides no runtime validation guarantees
  When valid: Internal data structures with no external validation requirements
- Use marshmallow schemas for validation and serialization (rejected)
  Rejected because: Marshmallow requires separate schema definitions and lacks the tight FastAPI integration that Pydantic provides for automatic dependency injection and response validation
  When valid: Projects already standardized on marshmallow or requiring more flexible serialization control
- Use TypedDict with runtime type checking libraries (rejected)
  Rejected because: TypedDict provides static type hints but no runtime validation, requiring additional libraries and integration work to achieve Pydantic's functionality
  When valid: Type hints for documentation purposes without runtime validation needs

## Risks

- Performance degradation in high-throughput endpoints due to Pydantic serialization overhead
  Mitigation: Profile critical paths and use Pydantic's performance mode or selective validation bypass for proven bottlenecks. Benchmark before and after Pydantic adoption.
  Owner: Engineering team
- Breaking changes in Pydantic v2 migration requiring codebase-wide updates
  Mitigation: Pin Pydantic major version in dependencies, establish migration timeline with automated codemods, and test migration in isolated services first
  Owner: Platform team
- Overly strict validation rejecting valid edge cases not anticipated in schema design
  Mitigation: Implement comprehensive integration tests covering edge cases, provide clear validation error messages, and establish exception process for legitimate use cases
  Owner: Engineering team

## Implementation Notes

- Use Pydantic Field with ge/le constraints for numeric ranges (e.g., scores from 0.0 to 1.0) as demonstrated in PhotoAnalysisOutput and GenZVibeCheckOutput
- Separate input models (Create, Update) from output models (Response) to control field mutability and prevent accidental exposure of internal fields
- Declare response_model on FastAPI router decorators to enable automatic response validation and OpenAPI schema generation
- Use descriptive Field descriptions to document business rules and constraints for automatic API documentation
- For complex validation requiring multiple fields, implement custom validators using Pydantic's @validator or @root_validator decorators

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l
- grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='
- python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')

Accept when:
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models

## Enforcement

- Verified by: Pre-commit hooks running mypy type checking on all Python files
- Verified by: CI pipeline grep checks verifying FastAPI endpoints declare response_model parameters
- Verified by: Code review checklist requiring Pydantic models for new service endpoints
- Violation handling: CI build fails if new FastAPI endpoints lack response_model declarations
- Violation handling: Code review blocks merge if validation logic duplicates Pydantic constraints
- Violation handling: Automated linting flags manual type coercion in handlers with Pydantic models
- Exception process: Submit exception request with performance benchmarks or migration timeline to tech lead
- Exception process: Document exception in ADR exceptions registry with approval date and review date
- Exception process: Exceptions reviewed quarterly to assess migration progress or permanent exemption