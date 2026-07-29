# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Handlers Raise

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all service boundary definitions in the monorepo/tlt domain, governing how HTTP endpoints expose business logic operations.

## Context

- The codebase contains multiple adapter services (discord_adapter) and core services (tlt_service) that expose business operations through HTTP interfaces using FastAPI framework
- Service boundaries are consistently defined using FastAPI APIRouter instances with HTTP method decorators (@router.post, @router.get, @router.delete) that map to domain operations
- Each service module (reminder.py, experience_manager.py, rsvp.py, monitor.py) follows a uniform pattern of declaring a router, defining Pydantic request/response models, and decorating async handler functions
- The pattern appears across 4 files with 92.22% confidence, indicating an established architectural convention for encoding business rules as HTTP service endpoints
- Services handle domain-specific operations (reminders, experiences, RSVPs, monitoring) with validation, error handling, and state management coordinated through the router boundary

## Problem Statement

Services need a consistent, type-safe mechanism to expose business logic operations as HTTP endpoints while maintaining clear boundaries between transport concerns and domain logic, ensuring validation, error handling, and API contracts are uniformly enforced across all service modules.

## Decision

1. SHOULD: Endpoint handlers SHOULD raise HTTPException with appropriate status codes (404, 500, 503) for error conditions rather than returning error responses

## Policy Block

- SHOULD Endpoint handlers SHOULD raise HTTPException with appropriate status codes (404, 500, 503) for error conditions rather than returning error responses

In scope:
- All HTTP service endpoints in monorepo/tlt/adapters
- All HTTP service endpoints in monorepo/tlt/services
- New service modules exposing business operations via HTTP
- Health check, monitoring, and administrative endpoints

Out of scope:
- Internal function calls between modules not exposed via HTTP
- Background tasks and scheduled jobs without HTTP triggers
- WebSocket or other non-HTTP protocol handlers
- CLI commands and scripts

Exceptions:
- EXC-001: Legacy endpoints being migrated from a different framework
- EXC-002: Endpoints requiring streaming responses or custom protocol handling incompatible with standard router decorators

## Rationale

- FastAPI's router-based architecture provides automatic OpenAPI schema generation, request validation, and response serialization, reducing boilerplate and preventing validation errors
- The pattern is observed consistently across 4 service modules (reminder.py, experience_manager.py, rsvp.py, monitor.py) with 92.22% confidence, indicating it is an established and proven convention
- Pydantic model integration ensures type safety and automatic validation at service boundaries, catching errors before they reach business logic
- Async handler functions enable efficient concurrent request processing, critical for services handling Discord events and real-time interactions

## Consequences

Positive:
- Automatic API documentation generation through OpenAPI/Swagger with minimal additional code
- Type-safe request/response handling with compile-time checking and runtime validation
- Consistent error handling patterns across all service endpoints using HTTPException
- Reduced boilerplate for common HTTP concerns (parsing, validation, serialization)
- Clear separation between transport layer (FastAPI routers) and business logic (domain operations)

Negative:
- Tight coupling to FastAPI framework makes migration to alternative frameworks costly
- Async function requirement increases complexity for simple synchronous operations
- Pydantic model definitions create additional code overhead for simple endpoints
- Router decorator syntax may be less familiar to developers from non-Python backgrounds

## Alternatives

- Use Flask with function-based views and manual validation (rejected)
  Rejected because: Lacks automatic validation, async support, and OpenAPI generation; requires significantly more boilerplate for equivalent functionality
  When valid: For simple synchronous services with minimal validation requirements
- Use Django REST Framework with class-based views (rejected)
  Rejected because: Heavier framework with ORM coupling; less suitable for microservices architecture; async support less mature than FastAPI
  When valid: For monolithic applications with complex ORM requirements and admin interfaces
- Use raw ASGI application with manual routing (rejected)
  Rejected because: Requires reimplementing validation, serialization, and routing logic; no automatic documentation; significantly higher development cost
  When valid: For performance-critical services where framework overhead is measurable bottleneck

## Risks

- FastAPI version upgrades may introduce breaking changes to router decorator syntax or Pydantic integration
  Mitigation: Pin FastAPI and Pydantic versions in requirements; test upgrades in isolated environment; maintain comprehensive integration tests
  Owner: Platform Engineering Team
- Inconsistent error handling across endpoints may leak internal implementation details or provide poor user experience
  Mitigation: Implement centralized exception handlers; establish error response schema standards; conduct code reviews focusing on error paths
  Owner: Service Development Teams
- Pydantic validation errors may expose sensitive information in error messages
  Mitigation: Configure Pydantic to sanitize error messages; implement custom validation error handlers; review validation logic for information disclosure
  Owner: Security Team

## Implementation Notes

- Create a router instance at module level: `router = APIRouter(prefix='/resource', tags=['Resource'])`
- Define Pydantic models for requests and responses before endpoint handlers, using type hints for all fields
- Use HTTPException for all error conditions with appropriate status codes: 400 (validation), 404 (not found), 500 (internal error), 503 (service unavailable)
- Include response_model parameter in router decorators to enable automatic validation and OpenAPI schema generation
- Register routers with the main FastAPI application using `app.include_router(router)` in application initialization

## Continuation Context


Verify commands:
- grep -r '@router\.(get|post|put|delete|patch)' monorepo/tlt --include='*.py' | wc -l
- grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l
- grep -r 'async def.*router\.' monorepo/tlt --include='*.py' | wc -l
- python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); print(any(isinstance(d, ast.AsyncFunctionDef) for d in ast.walk(tree)))" monorepo/tlt/adapters/discord_adapter/reminder.py

Accept when:
- All service endpoint handlers use FastAPI router decorators (@router.get, @router.post, etc.)
- All request and response data structures are defined as Pydantic BaseModel subclasses
- All endpoint handlers are async functions
- Router decorators specify response_model parameter for structured responses

## Enforcement

- Verified by: Automated code review checks scanning for router decorator patterns
- Verified by: CI pipeline linting rules enforcing async function signatures for endpoints
- Verified by: Type checking with mypy validating Pydantic model usage
- Verified by: Manual code review checklist items for service boundary definitions
- Violation handling: CI pipeline fails if endpoints lack proper router decorators or Pydantic models
- Violation handling: Code review blocks merge if service boundaries do not follow pattern
- Violation handling: Architecture review required for any exceptions to the pattern
- Violation handling: Existing violations tracked in technical debt backlog with remediation timeline
- Exception process: Submit exception request to architecture review board with technical justification
- Exception process: Document specific constraints preventing standard pattern adoption
- Exception process: Provide alternative approach with equivalent validation and documentation guarantees
- Exception process: Obtain approval from technical lead and document in ADR exceptions registry