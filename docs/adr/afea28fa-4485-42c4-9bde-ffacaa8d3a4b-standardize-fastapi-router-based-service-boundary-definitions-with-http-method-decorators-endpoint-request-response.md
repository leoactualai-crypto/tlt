# Standardize FastAPI Router-Based Service Boundary Definitions with HTTP Method Decorators: Endpoint Request Response

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all FastAPI-based service implementations within the monorepo.

## Context

- The monorepo contains multiple FastAPI-based service adapters (discord_adapter) and services (tlt_service) that expose HTTP endpoints for external clients and monitoring systems.
- Service boundaries are consistently defined using FastAPI APIRouter instances with explicit HTTP method decorators (@router.post, @router.get, @router.delete) and Pydantic BaseModel validation for request/response contracts.
- Four files demonstrate uniform application of this pattern across reminder management, experience tracking, RSVP handling, and service monitoring capabilities, indicating an established architectural convention.
- The pattern co-occurs with security.input_validation, domain.validation, and api.public.contracts facets, suggesting service boundary definitions are part of a broader secure coding and API contract enforcement strategy.
- Services integrate with external systems (Discord) and require structured health checks, metrics endpoints, and CRUD operations with consistent error handling via HTTPException.

## Problem Statement

Without standardized service boundary definitions, HTTP endpoints may lack consistent input validation, response contracts, error handling, and discoverability, increasing the risk of security vulnerabilities, API contract violations, and operational monitoring gaps across distributed service components.

## Decision

1. MUST: All endpoint request and response payloads MUST be validated using Pydantic BaseModel subclasses with explicit type annotations.

## Policy Block

- MUST All endpoint request and response payloads MUST be validated using Pydantic BaseModel subclasses with explicit type annotations.

In scope:
- All FastAPI-based HTTP services within the monorepo
- Discord adapter endpoints (reminder.py, experience_manager.py, rsvp.py)
- TLT service monitoring endpoints (monitor.py)
- Any new service modules exposing REST APIs or webhooks
- Health check and metrics endpoints for operational monitoring

Out of scope:
- Internal function calls and method invocations not exposed via HTTP
- gRPC or GraphQL service definitions using different frameworks
- WebSocket endpoints (unless using FastAPI WebSocket routes)
- CLI tools and batch processing scripts without HTTP interfaces
- Third-party library code outside the monorepo

Exceptions:
- EXC-001: Legacy endpoints undergoing migration may temporarily use unstructured response models during a transition period not exceeding one sprint cycle.
- EXC-002: Proxy or passthrough endpoints that forward external API responses may omit Pydantic response_model if the external schema is not controlled by the service.

## Rationale

- The pattern appears consistently across 4 files with 92.22% confidence, indicating established practice rather than isolated implementation.
- Co-occurrence with security.input_validation and domain.validation facets demonstrates that service boundary definitions are integral to the secure coding strategy, preventing injection attacks and data corruption.
- FastAPI's automatic OpenAPI schema generation from router definitions and Pydantic models provides self-documenting APIs, reducing integration errors and improving developer experience.
- Structured error handling via HTTPException enables consistent error responses across services, simplifying client-side error handling and operational debugging.

## Consequences

Positive:
- Automatic request/response validation prevents invalid data from entering service logic, reducing security vulnerabilities and runtime errors.
- Self-documenting APIs via OpenAPI schema generation improve discoverability and reduce integration time for internal and external consumers.
- Consistent error handling patterns simplify client implementation and operational troubleshooting across distributed services.
- Health check standardization enables uniform monitoring and alerting infrastructure across all service components.

Negative:
- Pydantic model definitions add boilerplate code and maintenance overhead when API contracts evolve frequently.
- Strict validation may reject edge cases or legacy data formats, requiring additional exception handling or data migration.
- FastAPI framework coupling increases migration cost if future architectural decisions require different HTTP frameworks.
- Async endpoint handlers require careful management of event loops and blocking operations to avoid performance degradation.

## Alternatives

- Use Flask with manual request validation and JSON schema validation libraries (rejected)
  Rejected because: Flask lacks automatic OpenAPI generation and requires manual validation code, increasing security risk and maintenance burden compared to FastAPI's integrated Pydantic validation.
  When valid: Valid for legacy services where migration cost exceeds benefit or when synchronous WSGI deployment is required.
- Implement gRPC service definitions with Protocol Buffers for strongly-typed contracts (rejected)
  Rejected because: gRPC adds complexity for browser-based clients and monitoring tools that expect REST/HTTP interfaces, and the existing Discord integration requires HTTP webhooks.
  When valid: Valid for high-performance internal service-to-service communication where HTTP overhead is prohibitive.
- Use GraphQL with schema-first design for flexible query capabilities (rejected)
  Rejected because: GraphQL adds complexity for simple CRUD operations and the current use cases do not require flexible querying or nested resource fetching.
  When valid: Valid for client-facing APIs with complex data relationships and variable query requirements.

## Risks

- Pydantic validation performance overhead may impact high-throughput endpoints processing thousands of requests per second.
  Mitigation: Profile endpoint performance under load testing and consider caching validated models or using Pydantic's compiled mode for hot paths.
  Owner: Engineering team
- Breaking changes to Pydantic models can cascade to all API consumers without versioning strategy.
  Mitigation: Implement API versioning via URL path prefixes (e.g., /v1/, /v2/) and maintain backward compatibility for at least two versions.
  Owner: API platform team
- Inconsistent HTTPException usage across services may lead to unpredictable error responses for clients.
  Mitigation: Create shared exception handler middleware and document standard error response format in API guidelines.
  Owner: Engineering team

## Implementation Notes

- Create a shared base router configuration module that includes common middleware, exception handlers, and CORS settings to ensure consistency across services.
- Establish Pydantic model naming conventions: suffix request models with 'Create', 'Update', or 'Request', and response models with 'Response' for clarity.
- Implement health check endpoints using a standard response format including timestamp, service name, status enum (healthy/degraded/unhealthy), and relevant metrics.
- Use FastAPI dependency injection for shared validation logic, authentication, and database session management to reduce code duplication across endpoints.
- Document all endpoints with docstrings that appear in OpenAPI schema, including parameter descriptions, example requests/responses, and error conditions.

## Continuation Context


Verify commands:
- grep -r '@router\.(get\|post\|delete\|put\|patch)' monorepo/tlt --include='*.py' | wc -l
- grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(Request|Response|Create|Update)' | wc -l
- grep -r 'response_model=' monorepo/tlt --include='*.py' | wc -l
- grep -r 'raise HTTPException' monorepo/tlt --include='*.py' | wc -l
- grep -r '@router\.get.*"/health"' monorepo/tlt --include='*.py'

Accept when:
- All service modules with HTTP endpoints contain at least one FastAPI APIRouter instance with decorated endpoint methods.
- All endpoint handlers that accept structured input or return structured output use Pydantic BaseModel subclasses for validation.
- All services exposing HTTP endpoints implement a /health endpoint returning structured status information.
- Grep commands for router decorators, BaseModel definitions, response_model usage, and HTTPException patterns return non-zero counts indicating pattern adoption.

## Enforcement

- Verified by: Pre-commit hooks running ruff or pylint with custom rules checking for router decorator usage and Pydantic model validation
- Verified by: CI pipeline static analysis scanning for endpoints without response_model declarations or missing HTTPException error handling
- Verified by: Code review checklist requiring verification of Pydantic models, health endpoints, and structured error handling for all new service endpoints
- Verified by: Automated OpenAPI schema validation ensuring all endpoints generate valid schema definitions
- Violation handling: CI pipeline fails if new endpoints are added without Pydantic validation or response_model declarations
- Violation handling: Code review blocks merge requests that introduce unvalidated endpoints or unstructured error responses
- Violation handling: Quarterly architecture audits identify non-compliant endpoints and create remediation tickets with priority based on security risk
- Violation handling: Runtime monitoring alerts on endpoints returning non-standard error formats or missing health check implementations
- Exception process: Submit exception request to architecture review board with justification, affected endpoints, and remediation timeline
- Exception process: Tech lead approval required for temporary exceptions during migration periods, with maximum duration of one sprint
- Exception process: Document approved exceptions in endpoint docstrings and maintain exception registry in architecture documentation
- Exception process: Review all active exceptions quarterly and require re-approval or remediation plan updates