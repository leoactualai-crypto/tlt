# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Models Use Optional

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all API boundary definitions and domain model validation within the codebase.

## Context

- The codebase contains multiple FastAPI service adapters (Discord adapter, TLT service) that expose HTTP endpoints requiring structured request/response validation
- Domain entities such as reminders, experiences, reactions, tasks, and photo analysis outputs require type-safe serialization and deserialization across API boundaries
- Input validation must occur at API ingress points to prevent invalid data from entering business logic layers
- The system integrates with external clients (Discord bot, HTTP clients) that require consistent contract definitions
- Pydantic BaseModel provides runtime type checking, automatic validation, and JSON schema generation aligned with FastAPI's native integration

## Problem Statement

API boundaries in distributed services require a standardized mechanism for validating incoming requests, serializing domain models, and enforcing type constraints at runtime. Without a consistent validation paradigm, services risk accepting malformed data, producing inconsistent responses, and creating maintenance burden through manual validation logic scattered across endpoints.

## Decision

1. MAY: Models MAY use Optional types for nullable fields with explicit default values

## Policy Block

- MAY Models MAY use Optional types for nullable fields with explicit default values

In scope:
- All FastAPI router endpoints in adapters (discord_adapter, service APIs)
- Domain models representing business entities (Reminder, Experience, Task, Photo analysis outputs)
- Request/response contracts for external HTTP clients
- Input validation at API ingress boundaries

Out of scope:
- Internal function signatures not crossing API boundaries
- Database ORM models (unless also serving as API contracts)
- Configuration objects loaded from environment variables
- Logging and observability data structures

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily use dict-based validation

## Rationale

- Evidence shows consistent pattern across 5 files with 92.22% confidence, indicating established architectural practice rather than isolated usage
- Pydantic BaseModel integration with FastAPI provides automatic OpenAPI schema generation, reducing documentation drift between implementation and API contracts
- Field-level constraints (Field(ge=0.0, le=1.0), rating: int # 1-5) enforce domain invariants at deserialization time, preventing invalid state propagation
- Separation of Create/Response model pairs (ReminderCreate/ReminderResponse, ExperienceCreate/ExperienceResponse) follows CQRS-lite pattern, distinguishing input validation from output serialization concerns

## Consequences

Positive:
- Automatic request validation at API boundaries reduces boilerplate validation code and centralizes error handling
- Type-safe domain models enable IDE autocomplete and static analysis, improving developer experience
- FastAPI integration generates OpenAPI schemas automatically, ensuring API documentation stays synchronized with implementation
- Runtime validation catches type errors and constraint violations before data enters business logic

Negative:
- Pydantic dependency couples domain models to validation framework, creating migration cost if validation strategy changes
- Runtime validation overhead adds latency to request processing (typically negligible but measurable at scale)
- Complex validation logic may require custom validators, increasing model complexity
- Tight coupling between API contracts and domain models can make schema evolution more difficult

## Alternatives

- Use dataclasses with manual validation logic in endpoint handlers (rejected)
  Rejected because: Requires scattered validation code across endpoints, no automatic OpenAPI generation, loses FastAPI native integration benefits
  When valid: Suitable for internal-only services with no external API contracts
- Use marshmallow schemas for serialization/deserialization (rejected)
  Rejected because: Marshmallow is not natively integrated with FastAPI, requires additional adapter code, less type-safe than Pydantic
  When valid: Valid for Flask-based services or when migrating from existing marshmallow codebase
- Use TypedDict with runtime type checking libraries (typeguard, beartype) (rejected)
  Rejected because: TypedDict provides static typing only, lacks field-level constraint validation, no automatic serialization
  When valid: Appropriate for internal type hints without validation requirements

## Risks

- Pydantic version upgrades may introduce breaking changes in validation behavior or API
  Mitigation: Pin Pydantic major version in dependencies, test validation behavior in CI, review Pydantic changelog before upgrades
  Owner: Engineering team
- Complex nested models may create performance bottleneck in high-throughput endpoints
  Mitigation: Profile validation overhead in performance tests, consider validation caching or lazy validation for read-heavy endpoints
  Owner: Engineering team
- Domain models tightly coupled to API contracts may hinder independent evolution of internal vs external schemas
  Mitigation: Introduce explicit mapping layer between domain entities and API models when schemas diverge significantly
  Owner: Architecture team

## Implementation Notes

- Import BaseModel from pydantic and define models with explicit type annotations: class ModelName(BaseModel): field: type
- Use Field() for constraints: rating: int = Field(ge=1, le=5, description='Rating from 1 to 5')
- Declare response_model in router decorators: @router.post('/', response_model=ResponseModel)
- Separate input models (Create, Update) from output models (Response) to distinguish validation concerns
- Use Optional[Type] with default None for nullable fields: photos: Optional[List[str]] = None

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services | wc -l
- grep -r '@router\.(get|post|put|delete).*response_model=' --include='*.py' monorepo/tlt | wc -l
- python -c 'import ast; import sys; tree = ast.parse(open(sys.argv[1]).read()); print(any(base.id == "BaseModel" for node in ast.walk(tree) if isinstance(node, ast.ClassDef) for base in node.bases if hasattr(base, "id")))' <file_path>

Accept when:
- All FastAPI endpoint files contain at least one BaseModel subclass for request or response validation
- All router endpoint decorators with complex request/response bodies declare response_model parameter
- Grep for 'class.*BaseModel' in API adapter directories returns count matching number of domain validation models

## Enforcement

- Verified by: Code review checklist requiring Pydantic models for new API endpoints
- Verified by: CI linting step using grep to verify BaseModel usage in router files
- Verified by: Static analysis with mypy to enforce type annotations on model fields
- Violation handling: CI pipeline fails if new router endpoints lack response_model declarations
- Violation handling: Code review blocks merge if validation models use untyped dictionaries
- Violation handling: Architecture review required for endpoints bypassing Pydantic validation
- Exception process: Document exception rationale in ADR exception log with EXC-ID reference
- Exception process: Obtain tech lead approval with written justification
- Exception process: Set expiration date for temporary exceptions with migration plan