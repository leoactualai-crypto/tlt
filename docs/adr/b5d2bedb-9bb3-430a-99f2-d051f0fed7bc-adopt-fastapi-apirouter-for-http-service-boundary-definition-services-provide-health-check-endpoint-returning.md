# Adopt FastAPI APIRouter for HTTP Service Boundary Definition: Services Provide Health Check Endpoint Returning

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The project implements multiple service boundaries including Discord adapters (reminder, RSVP) and MCP services (event manager, RSVP) that expose HTTP APIs to external clients and integration points
- Service endpoints require async request handling, automatic request/response validation, structured error handling, and OpenAPI documentation for client integration
- The architecture separates adapter layers from core business logic, requiring a consistent pattern for defining external API contracts at service boundaries
- Multiple services need modular route organization that can be composed into larger applications while maintaining clear separation of concerns
- The codebase uses Python async/await patterns throughout, requiring a framework that provides native async support for HTTP handlers

## Problem Statement

The system requires a consistent, type-safe approach to defining HTTP service boundaries across adapters and MCP services. Service endpoints must handle async operations, validate requests and responses against contracts, provide structured error handling, and generate API documentation automatically. Without a standardized pattern, service boundary implementations would diverge, making integration harder and reducing contract enforcement at runtime.

## Decision

1. SHOULD: Services SHOULD provide a health check endpoint returning service status, timestamp, and relevant metrics

## Policy Block

- SHOULD Services SHOULD provide a health check endpoint returning service status, timestamp, and relevant metrics

In scope:
- All adapter layer modules that expose HTTP endpoints to external clients
- All MCP service modules that define REST API boundaries
- Any module implementing service-to-service HTTP communication interfaces
- Health check and monitoring endpoints for operational visibility

Out of scope:
- Internal function calls within a service that do not cross service boundaries
- WebSocket or streaming endpoints that require different connection patterns
- Background tasks or scheduled jobs that do not expose HTTP interfaces
- CLI tools or scripts that interact with services as clients rather than defining service boundaries

## Rationale

- FastAPI provides native async/await support matching the codebase's concurrency model, enabling efficient handling of I/O-bound operations in Discord adapters and event management services
- The APIRouter pattern enables modular route organization observed across 5 files, allowing service boundaries to be defined independently and composed into larger applications
- Pydantic model integration enforces contract validation at runtime, catching type errors and invalid requests before they reach business logic, as evidenced by BaseModel usage in reminder and RSVP endpoints
- Automatic OpenAPI documentation generation supports client integration and API discovery without manual documentation maintenance, critical for MCP service interoperability

## Consequences

Positive:
- Consistent service boundary definition pattern across all adapters and MCP services reduces cognitive load and accelerates development
- Automatic request/response validation prevents invalid data from reaching business logic, improving system reliability
- Type annotations enable IDE autocomplete and static analysis, catching errors at development time
- Generated OpenAPI documentation stays synchronized with implementation, eliminating documentation drift

Negative:
- FastAPI dependency couples service boundary layer to a specific framework, requiring migration effort if framework needs change
- Decorator-based route registration can make dynamic route generation more complex compared to imperative registration
- Pydantic model overhead adds serialization/deserialization cost for high-throughput endpoints
- Framework learning curve required for developers unfamiliar with FastAPI's dependency injection and validation patterns

## Alternatives

- Use Flask with marshmallow for route definition and validation (rejected)
  Rejected because: Flask lacks native async support and requires additional libraries for async handlers; marshmallow validation is not integrated with type hints; no automatic OpenAPI generation
  When valid: For synchronous services with simple request/response patterns where async support is not required
- Use Django REST Framework for service boundaries (rejected)
  Rejected because: Django's ORM and full-stack framework overhead is unnecessary for service boundary layer; class-based views add complexity; async support is less mature than FastAPI
  When valid: For applications requiring Django's admin interface, ORM, and full-stack capabilities
- Use raw ASGI with manual route handling and validation (rejected)
  Rejected because: Requires manual implementation of routing, validation, error handling, and documentation; increases maintenance burden and error potential; no type-based validation
  When valid: For extremely performance-critical services where framework overhead is measured and unacceptable

## Risks

- FastAPI version updates may introduce breaking changes to decorator syntax or Pydantic integration
  Mitigation: Pin framework versions in dependency manifest; test upgrades in isolated environment before production deployment; monitor FastAPI changelog for breaking changes
  Owner: Backend engineering team
- Pydantic model validation overhead may impact latency for high-frequency endpoints
  Mitigation: Profile endpoint performance under load; use Pydantic's performance optimization features; consider validation bypass for trusted internal calls if measured impact is significant
  Owner: Backend engineering team
- Inconsistent error handling patterns across services may leak implementation details or provide inconsistent client experience
  Mitigation: Establish HTTPException usage conventions with standard status codes; implement centralized exception handlers for common error types; document error response schemas
  Owner: API platform team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Create one APIRouter instance per logical service module; use router prefixes to namespace related endpoints; compose routers into the main application using include_router
- Define Pydantic models in a shared models module when contracts are reused across multiple endpoints; keep endpoint-specific models colocated with route definitions
- Use response_model parameter in route decorators to enforce response validation and enable automatic OpenAPI schema generation; leverage status_code parameter for non-200 success responses
- Implement health check endpoints following the observed pattern: return status, timestamp, version, and service-specific metrics as a dictionary; use async def for consistency even if the handler performs no async operations

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the resolved version of the web framework; confirm the version matches the lock artifact
- Locate service boundary modules in adapter and MCP service directories; verify all route definitions use the decorator pattern with typed models
- Identify the project's test suite location; execute integration tests for service endpoints to verify contract validation and error handling behavior

Accept when:
- All HTTP service boundary modules import and instantiate APIRouter; all route handlers use method decorators with path and response_model parameters
- All request and response contracts are defined as Pydantic BaseModel subclasses with typed fields; validation errors return appropriate HTTP 422 responses
- Integration tests pass for all service endpoints demonstrating request validation, response serialization, and error handling via HTTPException

## Enforcement

- Verified by: Code review checklist verifying APIRouter usage and Pydantic model contracts for all new service endpoints
- Verified by: Static analysis scanning for route handler patterns and type annotations on endpoint functions
- Verified by: Integration test suite coverage requirements for all service boundary endpoints
- Violation handling: Code review blocks merge if service endpoints do not follow APIRouter decorator pattern
- Violation handling: CI pipeline fails if integration tests are missing for new service boundary endpoints
- Violation handling: Architecture review required for any service boundary implementation using alternative frameworks or patterns
- Exception process: Document technical justification for alternative pattern including performance measurements or framework limitations
- Exception process: Obtain approval from backend architecture team lead
- Exception process: Create follow-up task to evaluate whether exception pattern should become standard or be migrated to conform