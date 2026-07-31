# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Basemodel Classes Define

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all internal API endpoints within the monorepo that handle structured request/response data.

## Context

- Internal APIs across Discord adapters (reminder.py, rsvp.py, event.py), MCP services (photo_processor.py), and TLT services (event_manager.py) require structured data validation for request payloads and response contracts
- FastAPI router endpoints expose HTTP interfaces that accept user-provided data requiring type safety, constraint validation, and automatic serialization/deserialization
- Domain models such as ReminderCreate, EventCreate, ReactionUpdate, PhotoAnalysisOutput, GenZVibeCheckOutput, and TaskResponse define explicit schemas with field-level constraints using Pydantic Field descriptors
- The codebase demonstrates consistent adoption of Pydantic BaseModel across 5 files with 92% confidence, indicating an established pattern rather than isolated usage
- Integration testing patterns show router endpoints declaring response_model parameters that enforce output schema validation at the framework level

## Problem Statement

Internal APIs require a standardized mechanism to validate incoming request data, enforce type safety, apply field-level constraints (e.g., ge=0.0, le=1.0), and serialize response objects with guaranteed schema compliance. Without a consistent validation layer, endpoints risk accepting malformed data, returning inconsistent response structures, and creating implicit contracts that are difficult to test, document, or evolve safely.

## Decision

1. MAY: BaseModel classes MAY define custom validators using Pydantic @validator decorators for cross-field validation logic

## Policy Block

- MAY BaseModel classes MAY define custom validators using Pydantic @validator decorators for cross-field validation logic

In scope:
- All FastAPI router endpoints in Discord adapters (reminder.py, rsvp.py, event.py)
- All FastAPI router endpoints in TLT services (event_manager.py)
- All structured data contracts in MCP services (photo_processor.py) that interface with external systems
- Request models for POST, PUT, PATCH operations accepting JSON payloads
- Response models for all endpoints returning structured data (not raw strings or binary)

Out of scope:
- External third-party API clients where response schemas are not controlled by this codebase
- Internal utility functions that do not expose HTTP endpoints
- CLI tools or scripts that accept command-line arguments rather than structured payloads
- WebSocket or streaming endpoints where message schemas are defined by protocol specifications
- Health check endpoints returning simple status dictionaries without domain semantics

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily accept Dict[str, Any] with explicit validation logic until BaseModel migration is complete
- EXC-002: Proxy endpoints forwarding opaque payloads to downstream services may skip BaseModel validation if payload structure is not interpreted

## Rationale

- Evidence shows consistent adoption of Pydantic BaseModel across 5 files (reminder.py, rsvp.py, event.py, photo_processor.py, event_manager.py) with 92% confidence, indicating an established architectural pattern
- FastAPI framework integration with Pydantic enables automatic request validation, response serialization, and OpenAPI schema generation without additional boilerplate
- Field-level constraints (e.g., ge=0.0, le=1.0 in PhotoAnalysisOutput, GenZVibeCheckOutput) enforce domain invariants at the API boundary, preventing invalid data from entering business logic
- Explicit response_model declarations in router decorators provide compile-time type safety and runtime validation, reducing the risk of schema drift between implementation and documentation

## Consequences

Positive:
- Automatic request validation with detailed HTTP 422 error responses reduces manual validation code and improves API consumer debugging experience
- Type-safe request/response contracts enable IDE autocomplete, static analysis, and refactoring tools to detect schema mismatches at development time
- OpenAPI schema generation from BaseModel definitions ensures API documentation remains synchronized with implementation
- Field-level constraints (ge, le, Field descriptors) encode domain invariants declaratively, making validation logic explicit and testable in isolation

Negative:
- Pydantic validation overhead adds latency to request processing, particularly for deeply nested models or large array payloads
- BaseModel classes introduce additional code surface area and maintenance burden compared to untyped dictionaries
- Complex validation logic requiring cross-field dependencies or external data lookups may not fit cleanly into Pydantic validator patterns, requiring custom validation layers
- Schema evolution (adding/removing fields) requires careful coordination with API consumers to avoid breaking changes, particularly for required fields

## Alternatives

- Use untyped Dict[str, Any] with manual validation logic in endpoint handlers (rejected)
  Rejected because: Manual validation is error-prone, lacks automatic OpenAPI schema generation, and creates implicit contracts that are difficult to test and document. Evidence shows this pattern was abandoned in favor of BaseModel across all observed files.
  When valid: Only valid for proxy endpoints forwarding opaque payloads without interpretation
- Use dataclasses with separate validation library (e.g., marshmallow, cerberus) (rejected)
  Rejected because: Requires additional dependency and manual integration with FastAPI. Pydantic is FastAPI's native validation layer with zero-configuration integration for request/response handling and OpenAPI generation.
  When valid: Valid for non-FastAPI components where Pydantic dependency is undesirable
- Use JSON Schema with runtime validation via jsonschema library (rejected)
  Rejected because: JSON Schema provides validation but lacks type safety, IDE support, and automatic serialization. Pydantic generates JSON Schema from Python types, providing both compile-time and runtime guarantees.
  When valid: Valid for polyglot systems where schema must be shared across non-Python services

## Risks

- Pydantic version upgrades may introduce breaking changes in validation behavior or Field API, requiring codebase-wide updates
  Mitigation: Pin Pydantic major version in requirements.txt, test validation behavior in CI, and review Pydantic changelog before upgrades
  Owner: Platform Engineering Team
- Complex nested models with deep validation may introduce performance bottlenecks in high-throughput endpoints
  Mitigation: Profile validation overhead in load tests, consider lazy validation or schema simplification for performance-critical paths, and document performance characteristics in endpoint docstrings
  Owner: API Development Team
- Schema evolution (adding required fields) creates breaking changes for existing API consumers without versioning strategy
  Mitigation: Adopt API versioning strategy (URL path or header-based), use optional fields with defaults for backward compatibility, and document deprecation timelines
  Owner: API Governance Team

## Implementation Notes

- Define BaseModel classes in dedicated schema modules (e.g., schemas.py) separate from router definitions to enable reuse across endpoints and testing
- Use Pydantic Field descriptors with description parameters to generate self-documenting OpenAPI schemas: Field(description='User ID from Discord', ge=1)
- Leverage FastAPI response_model parameter in router decorators to enforce output validation: @router.post('/', response_model=ReminderResponse)
- For complex validation logic, use Pydantic @validator decorators with pre=True or post=True to control validation order and access to other fields
- Test BaseModel validation in isolation using pytest fixtures that instantiate models with valid/invalid data, separate from integration tests of router endpoints

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | wc -l
- grep -r '@router\.(post|put|patch).*response_model=' monorepo/tlt/adapters monorepo/tlt/services --include='*.py' | wc -l
- grep -r 'Field(' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | grep -E '(ge=|le=|min_length=|max_length=|description=)' | wc -l

Accept when:
- All FastAPI router endpoints accepting structured request bodies define corresponding BaseModel subclasses with explicit field types
- All FastAPI router endpoints returning structured responses declare response_model parameter with BaseModel subclass
- BaseModel classes with numeric or string fields include Field descriptors with appropriate constraints (ge, le, min_length, etc.) where domain invariants exist

## Enforcement

- Verified by: CI pipeline runs grep-based verification commands to count BaseModel usage and response_model declarations
- Verified by: Code review checklist requires reviewers to verify new endpoints define BaseModel request/response schemas
- Verified by: Static analysis with mypy enforces type annotations on router endpoint parameters and return types
- Violation handling: CI build fails if new router endpoints are detected without corresponding BaseModel definitions
- Violation handling: Code review blocks merge if endpoints accept Dict[str, Any] without documented exception approval
- Violation handling: Quarterly architecture review audits endpoints for schema drift and missing Field constraints
- Exception process: Developer submits exception request to tech lead with justification (legacy migration, proxy semantics, performance constraints)
- Exception process: Tech lead reviews request and approves with documented timeline or alternative approach
- Exception process: Exception is recorded in endpoint docstring with EXC-ID reference and target resolution date
- Exception process: Exceptions are reviewed quarterly and expired exceptions trigger refactoring tasks