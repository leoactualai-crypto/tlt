# Standardize FastAPI Router Decorators for Internal Service Endpoint Definitions: Internal Services Exposing

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all internal service implementations within the monorepo that expose HTTP endpoints.

## Context

- Multiple internal services (event_manager, rsvp, discord_adapter) expose HTTP endpoints using FastAPI router decorators with consistent patterns for health checks, redirects, and business logic endpoints
- Services require standardized endpoint definitions to support load balancer health checks, MCP protocol compatibility, and inter-service communication within the monorepo
- The codebase demonstrates a recurring pattern of using @router.get(), @router.post(), @router.put(), @router.delete() decorators with response_model specifications and async handlers
- Health check endpoints (/health, /ping) and redirect handlers (mcp_redirect_fix) appear across multiple services, indicating a need for consistent service boundary definitions
- Services use loguru for logging, FastMCP for MCP protocol support, and environment-based configuration (ENV, LOG_LEVEL) requiring standardized initialization patterns

## Problem Statement

Internal services within the monorepo lack a formalized standard for defining service boundaries and HTTP endpoints, leading to inconsistent patterns across event_manager, rsvp, and discord_adapter services. Without explicit guidance on router decorator usage, response model specifications, and health check implementations, services may diverge in their API contracts, making inter-service communication and operational monitoring more difficult to maintain.

## Decision

1. MUST: All internal services exposing HTTP endpoints MUST use FastAPI APIRouter instances with explicit decorator patterns (@router.get, @router.post, @router.put, @router.delete)

## Policy Block

- MUST All internal services exposing HTTP endpoints MUST use FastAPI APIRouter instances with explicit decorator patterns (@router.get, @router.post, @router.put, @router.delete)

In scope:
- All Python-based internal services within the monorepo/tlt directory structure
- MCP services (event_manager, rsvp) and adapter services (discord_adapter)
- HTTP endpoints exposed for inter-service communication, health checks, and external client integration
- FastAPI-based service implementations using APIRouter for endpoint definitions

Out of scope:
- External public APIs exposed outside the monorepo boundary
- Non-HTTP service interfaces (gRPC, message queues, database connections)
- Third-party library endpoints not under direct service control
- Static file serving or frontend routing patterns

Exceptions:
- EXC-001: Legacy services undergoing migration may temporarily use synchronous (non-async) handlers during transition period
- EXC-002: Services with specialized logging requirements may use alternative logging frameworks if loguru cannot meet operational needs

## Rationale

- Evidence shows 5 files across 3 distinct services (event_manager, rsvp, discord_adapter) consistently using FastAPI router decorators with async handlers, indicating an established pattern worth formalizing
- Health check endpoints appear in multiple services with similar structure, demonstrating operational requirements for service monitoring and load balancer integration
- The pattern achieves 91.62% confidence across detected instances, suggesting strong consistency in implementation approach that benefits from explicit standardization
- Standardizing on FastAPI router patterns enables type-safe API contracts through Pydantic response models, improving inter-service communication reliability and reducing runtime errors

## Consequences

Positive:
- Consistent service boundary definitions improve developer onboarding and reduce cognitive load when working across multiple services
- Standardized health check endpoints enable uniform operational monitoring and automated service discovery
- Type-safe response models through Pydantic reduce runtime errors and improve API contract documentation
- Async handler pattern supports efficient concurrent request processing and better resource utilization

Negative:
- Requires FastAPI as a mandatory dependency for all internal services, limiting framework flexibility
- Async handler requirement increases complexity for simple synchronous operations that don't benefit from concurrency
- Pydantic response model overhead may be unnecessary for simple endpoints returning primitive types or status codes
- Standardization may slow adoption of alternative patterns that could be more appropriate for specific service requirements

## Alternatives

- Use Flask with synchronous handlers for simpler service implementations (rejected)
  Rejected because: Evidence shows established FastAPI adoption across multiple services with async patterns; Flask would introduce framework fragmentation and lose type-safety benefits of Pydantic integration
  When valid: Could be reconsidered for extremely simple services with no concurrency requirements and minimal API surface
- Allow each service to define custom health check endpoint paths and response formats (rejected)
  Rejected because: Operational tooling (load balancers, monitoring systems) requires consistent health check endpoints; custom paths increase configuration complexity and reduce reliability
  When valid: Not recommended; standardization provides clear operational benefits
- Generate OpenAPI specifications from code and use code generation for client libraries (deferred)
  Rejected because: Not mutually exclusive with this ADR; can be adopted as complementary tooling after endpoint patterns are standardized
  When valid: Should be evaluated once service endpoint patterns are stable and consistent

## Risks

- Existing services not following the pattern may require significant refactoring, potentially introducing bugs during migration
  Mitigation: Implement gradual migration with comprehensive testing; use policy exceptions for services in transition; prioritize high-traffic services for early adoption
  Owner: Platform Engineering Team
- FastAPI version updates may introduce breaking changes to router decorator behavior or async handling
  Mitigation: Pin FastAPI version in dependency management; establish testing suite for endpoint patterns; document upgrade procedures and compatibility requirements
  Owner: Platform Engineering Team
- Async handler requirement may lead to blocking operations in async context, degrading performance
  Mitigation: Provide clear guidelines and examples for handling blocking I/O in async handlers; implement linting rules to detect blocking calls; offer training on async patterns
  Owner: Engineering Team

## Implementation Notes

- Create a shared base router module in tlt/common/api/ providing standard health check and ping endpoint implementations that services can import and register
- Establish Pydantic base models for common response patterns (health checks, error responses) to ensure consistency across services
- Document async handler patterns with examples showing proper handling of database queries, external API calls, and CPU-bound operations
- Provide service template or cookiecutter project structure demonstrating correct FastAPI router setup, logging configuration, and endpoint organization
- Consider implementing a shared middleware for request logging, error handling, and metrics collection that works consistently across all router-based services

## Continuation Context


Verify commands:
- grep -r '@router\.(get|post|put|delete)' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v 'async def' && echo 'FAIL: Found non-async handlers' || echo 'PASS: All handlers are async'
- grep -r 'def health_check\|def ping' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v '@router\.get' && echo 'FAIL: Health endpoints not using router.get' || echo 'PASS: Health endpoints properly decorated'
- python -c "import ast; import sys; files = ['monorepo/tlt/mcp_services/event_manager/main.py', 'monorepo/tlt/mcp_services/rsvp/main.py']; [sys.exit(1) for f in files if 'from loguru import logger' not in open(f).read()]; print('PASS: All services use loguru')"

Accept when:
- All service files in mcp_services/ and adapters/ directories use @router decorator patterns with async function definitions
- Every service implements /health endpoint returning JSON with status field and service-specific metrics
- Grep verification commands pass without detecting synchronous handlers or missing health endpoints

## Enforcement

- Verified by: Pre-commit hooks running grep patterns to detect non-async handlers and missing health endpoints
- Verified by: CI pipeline integration tests verifying /health and /ping endpoints respond with expected status codes
- Verified by: Code review checklist requiring FastAPI router pattern compliance for new services
- Verified by: Automated OpenAPI schema validation ensuring response_model specifications match actual handler return types
- Violation handling: CI pipeline fails if verification commands detect non-compliant endpoint definitions
- Violation handling: Code review blocks merge requests that introduce synchronous handlers without documented exception approval
- Violation handling: Monitoring alerts trigger if health check endpoints return unexpected response formats
- Violation handling: Quarterly architecture review identifies non-compliant services for prioritized refactoring
- Exception process: Submit exception request to architecture team with service name, specific rule requiring exception, and technical justification
- Exception process: Architecture team reviews within 5 business days, considering operational impact and migration timeline
- Exception process: Approved exceptions documented in service README with expiration date and migration plan
- Exception process: Exception status reviewed quarterly; expired exceptions without progress trigger escalation to engineering leadership