# Enforce Pydantic BaseModel Validation for Public API Contracts: Public Response Models

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public/external API endpoints and data contracts within the monorepo/tlt system. All request/response models exposed through FastAPI routers, MCP service interfaces, and inter-service boundaries MUST use Pydantic BaseModel validation.

## Context

- The monorepo/tlt system exposes multiple public API surfaces through FastAPI routers in adapters (discord_adapter) and MCP services (vibe_bit, guild_manager, photo_vibe_check), requiring consistent input validation across 10+ service endpoints
- External clients (Discord bots, event managers, photo processors) submit user-generated content including emoji placements, event RSVPs, guild registrations, and photo uploads that require type safety and constraint enforcement
- The system processes real-time user interactions with rate limiting, canvas placement rules, and time-based constraints that must be validated before business logic execution
- Multiple service boundaries exist between adapters, MCP services, and agents, creating integration points where data contracts must be explicitly defined and validated
- The codebase demonstrates consistent use of Pydantic BaseModel across all detected API contract definitions, establishing an existing architectural pattern

## Problem Statement

Public API endpoints accepting external input from Discord users, HTTP clients, and inter-service calls require type-safe validation to prevent invalid data from reaching business logic, causing runtime errors, security vulnerabilities, or data corruption. Without enforced schema validation at API boundaries, the system is vulnerable to malformed requests, type mismatches, constraint violations, and injection attacks.

## Decision

1. MUST: All public API response models MUST inherit from pydantic.BaseModel to ensure consistent serialization and type safety

## Policy Block

- MUST All public API response models MUST inherit from pydantic.BaseModel to ensure consistent serialization and type safety

In scope:
- All FastAPI router endpoints in monorepo/tlt/adapters/discord_adapter (event.py, rsvp.py, reminder.py)
- All MCP service models in monorepo/tlt/mcp_services (vibe_bit/models.py, guild_manager/models.py, photo_vibe_check/photo_processor.py)
- All service-to-service contract definitions used in inter-service communication
- All data models exposed through HTTP APIs, CloudEvents endpoints, or agent task interfaces
- Request/response models for batch task submission, event management, and guild operations

Out of scope:
- Internal domain models used only within service boundaries that are never exposed to external clients
- Database ORM models (SQLAlchemy, etc.) that have their own validation mechanisms
- Configuration objects loaded from environment variables or config files (unless exposed via API)
- Temporary data structures used in processing pipelines that are not part of API contracts
- Test fixtures and mock data objects used exclusively in test suites

Exceptions:
- EXC-001: Legacy endpoints being migrated from untyped dictionaries to Pydantic models
- EXC-002: Performance-critical paths where Pydantic validation overhead is measured and unacceptable

## Rationale

- Evidence shows 10 files across adapters and MCP services consistently using Pydantic BaseModel for API contracts (VibeElement, CanvasConfig, ReminderCreate, GuildRegistrationData, EventCreate, PhotoAnalysisOutput), establishing a proven pattern with 91.72% confidence
- Pydantic provides declarative validation with Field constraints (ge=32, le=1024 for canvas dimensions, ge=0.0, le=1.0 for scores) that prevent invalid data at the API boundary before business logic execution
- FastAPI's native integration with Pydantic enables automatic OpenAPI schema generation, request validation, and serialization without additional code, reducing maintenance burden
- Type-safe models with explicit validation reduce runtime errors, improve IDE support, and provide clear contracts for external clients integrating with Discord adapters and MCP services

## Consequences

Positive:
- Automatic request validation at API boundaries prevents invalid data from reaching business logic, reducing defensive programming and error handling code
- Type safety and IDE autocomplete improve developer experience when implementing or consuming APIs
- Automatic OpenAPI/JSON schema generation provides accurate API documentation without manual maintenance
- Consistent validation approach across all services reduces cognitive load and makes codebase more maintainable

Negative:
- Pydantic validation adds computational overhead to request processing, potentially impacting high-throughput endpoints
- Complex validation logic may become scattered between Field constraints, validators, and business logic if not carefully organized
- Model definitions can become verbose for complex nested structures with many constraints
- Breaking changes to models require careful versioning to avoid disrupting external clients

## Alternatives

- Use raw dictionaries with manual validation in endpoint handlers (rejected)
  Rejected because: Manual validation is error-prone, lacks type safety, requires repetitive code, and provides no automatic schema generation. Evidence shows this approach was abandoned in favor of Pydantic across all detected services.
  When valid: Never recommended for new code; only acceptable in legacy code being migrated
- Use dataclasses with separate validation libraries (marshmallow, cerberus) (rejected)
  Rejected because: Requires separate validation layer, lacks FastAPI integration, and introduces additional dependencies. Pydantic provides superior FastAPI integration and is already established in the codebase.
  When valid: Only if migrating from existing marshmallow-based services with significant investment
- Use TypedDict with runtime type checking (typeguard, beartype) (rejected)
  Rejected because: TypedDict provides static type hints but no runtime validation or constraint enforcement. Does not prevent invalid data from reaching business logic.
  When valid: Acceptable for internal type hints but insufficient for API boundary validation

## Risks

- Performance degradation on high-throughput endpoints due to Pydantic validation overhead
  Mitigation: Profile critical paths, use Pydantic's compiled mode, consider caching validated models, and document EXC-002 exception process for proven bottlenecks
  Owner: Service owners and performance engineering team
- Breaking changes to models disrupt external clients without proper versioning strategy
  Mitigation: Implement API versioning (URL path or header-based), maintain backward compatibility for at least one version, and provide migration guides for breaking changes
  Owner: API platform team and service owners
- Complex nested models become difficult to maintain and test as validation logic grows
  Mitigation: Decompose large models into smaller reusable components, use composition over inheritance, and maintain comprehensive test coverage for validation edge cases
  Owner: Engineering team and code reviewers

## Implementation Notes

- Use pydantic.Field with descriptive descriptions for automatic OpenAPI documentation generation (e.g., Field(description='Photo quality score from 0 to 1'))
- Leverage Field(default_factory=...) for datetime fields to avoid mutable default issues (e.g., Field(default_factory=lambda: datetime.now(timezone.utc)))
- Define Enum classes for fields with fixed value sets and reference them in model fields to enforce valid options at validation time
- Use Optional[T] typing for truly optional fields and provide sensible defaults with Field(default=...) to improve API usability
- Implement custom validators using @validator decorators for cross-field validation or complex business rules that cannot be expressed with Field constraints

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/*.py monorepo/tlt/mcp_services/*/models.py | wc -l
- grep -r '@router\.(post|put|patch)' monorepo/tlt/adapters/discord_adapter/*.py | xargs grep -L 'BaseModel'
- python -c 'import ast; import sys; [sys.exit(1) for f in sys.argv[1:] if not any(isinstance(n, ast.ClassDef) and any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name)) for n in ast.walk(ast.parse(open(f).read())))]' monorepo/tlt/adapters/discord_adapter/*.py

Accept when:
- All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models
- All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation
- No new API endpoints are merged that accept raw dictionaries or unvalidated input without documented exception approval
- Grep verification shows 100% of router endpoints with POST/PUT/PATCH methods reference BaseModel in the same file or imported models

## Enforcement

- Verified by: Pre-commit hooks running grep verification commands to detect unvalidated API endpoints
- Verified by: Code review checklist requiring Pydantic BaseModel usage for all new API endpoints
- Verified by: CI pipeline static analysis checking for FastAPI routes without BaseModel request/response models
- Verified by: Periodic architecture audits scanning for policy violations and exception compliance
- Violation handling: CI build fails if new API endpoints lack Pydantic validation without documented exception
- Violation handling: Code review blocks merge until BaseModel validation is added or exception is approved
- Violation handling: Existing violations are tracked in technical debt backlog with prioritized remediation plan
- Violation handling: Security review required for any endpoint handling user input without Pydantic validation
- Exception process: Submit exception request with justification (performance benchmarks or migration plan) to architecture review board
- Exception process: Provide alternative validation approach with equivalent security guarantees for performance exceptions
- Exception process: Document exception in endpoint code with EXC-001 or EXC-002 reference and approval date
- Exception process: Set remediation timeline for migration exceptions with quarterly progress reviews