# Enforce Pydantic BaseModel Validation for All Input Data Models: Models Include Field

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all services, agents, and adapters that process external input, API requests, event payloads, or user-submitted data. All data models accepting untrusted input MUST use Pydantic BaseModel validation.

## Context

- The codebase processes diverse external inputs including Discord events, CloudEvents, HTTP API requests, photo URLs, timer contexts, and user-submitted data across multiple MCP services and agents
- Input validation failures can lead to type errors, injection vulnerabilities, data corruption, and runtime exceptions that compromise service reliability and security
- Pydantic BaseModel provides declarative schema validation with Field constraints (ge, le, default_factory), type coercion, and automatic error reporting at deserialization boundaries
- Evidence shows consistent adoption of Pydantic BaseModel across 11 files spanning photo processing, gateway models, agent state, RSVP management, event management, and guild services
- The pattern establishes validation at trust boundaries before data enters business logic, preventing invalid states and reducing defensive programming overhead

## Problem Statement

Without systematic input validation at trust boundaries, services are vulnerable to malformed data, type mismatches, constraint violations, and injection attacks. Manual validation is error-prone, inconsistent across services, and creates maintenance burden. The system requires a standardized, declarative approach to validate all external input before it reaches business logic, ensuring type safety, constraint enforcement, and early error detection.

## Decision

1. SHOULD: Models SHOULD include Field descriptions for all fields to document validation intent and improve API documentation

## Policy Block

- SHOULD Models SHOULD include Field descriptions for all fields to document validation intent and improve API documentation

In scope:
- All MCP service models (gateway, RSVP, event_manager, photo_vibe_check, guild_manager)
- All agent state models (ambient_event_agent state, AgentTask, IncomingEvent)
- All adapter models (discord_adapter experience manager)
- API request/response models for HTTP endpoints
- CloudEvent and timer context models
- Configuration models loaded from external sources

Out of scope:
- Internal data transfer objects used only within a single module with trusted data
- Database ORM models that have separate validation layers
- Test fixtures and mock objects in test suites
- Temporary data structures in private helper functions

Exceptions:
- EXC-001: Performance-critical hot paths where validation overhead is measured and unacceptable
- EXC-002: Legacy integration points with third-party systems requiring raw data access

## Rationale

- Evidence shows 11 files across photo processing, gateway, agent state, RSVP, event management, and guild services consistently using Pydantic BaseModel with Field constraints for input validation
- Pydantic provides automatic type coercion, constraint validation (ge/le for numeric bounds), and clear error messages at deserialization time, preventing invalid data from entering business logic
- The pattern establishes validation at trust boundaries (API endpoints, event handlers, external data sources) where untrusted input enters the system, following defense-in-depth principles
- Declarative validation reduces code duplication, improves maintainability, and generates self-documenting schemas that can be used for API documentation and client code generation

## Consequences

Positive:
- Type safety and constraint enforcement at deserialization boundaries prevent invalid data from reaching business logic
- Automatic validation error messages improve debugging and API usability by providing clear feedback on validation failures
- Declarative schema definitions serve as living documentation and enable automatic OpenAPI/JSON Schema generation
- Reduced defensive programming overhead in business logic since validation guarantees are established at entry points

Negative:
- Pydantic validation adds runtime overhead at deserialization time, which may impact performance in high-throughput scenarios
- Complex validation logic may require custom validators, increasing model complexity and testing requirements
- Tight coupling to Pydantic library creates migration cost if validation framework needs to change
- Validation errors at API boundaries may expose internal schema details in error messages if not properly sanitized

## Alternatives

- Manual validation using isinstance checks and conditional logic in business logic functions (rejected)
  Rejected because: Manual validation is error-prone, inconsistent across services, creates code duplication, and lacks automatic error reporting. Evidence shows no usage of this pattern in the codebase.
  When valid: Never recommended for new code; only acceptable in legacy code scheduled for refactoring
- JSON Schema validation with jsonschema library for declarative validation (rejected)
  Rejected because: JSON Schema lacks Python type integration, requires separate schema files, does not provide type hints for IDEs, and has weaker ecosystem integration compared to Pydantic
  When valid: Only when interoperating with systems that require JSON Schema format for contract validation
- Dataclasses with manual validation in __post_init__ methods (rejected)
  Rejected because: Dataclasses lack built-in validation, constraint enforcement, and automatic error reporting. Validation logic in __post_init__ is less declarative and harder to maintain.
  When valid: Acceptable for internal data structures with trusted data sources where validation overhead is unnecessary

## Risks

- Performance degradation in high-throughput endpoints due to Pydantic validation overhead
  Mitigation: Profile validation performance in critical paths, use Pydantic's compiled mode (pydantic-core), and consider caching validated models for repeated operations
  Owner: Engineering team with performance testing responsibility
- Validation error messages may leak internal schema details or sensitive information to external clients
  Mitigation: Implement error sanitization layer at API boundaries that transforms Pydantic ValidationError into generic client-safe error responses
  Owner: Security team and API gateway maintainers
- Inconsistent validation coverage if developers bypass BaseModel for convenience or performance
  Mitigation: Enforce validation requirements through code review, linting rules (e.g., custom pylint checks), and automated testing that verifies all API endpoints use validated models
  Owner: Architecture review board and CI/CD pipeline maintainers

## Implementation Notes

- Use Field(ge=0.0, le=1.0) for normalized scores and percentages, Field(ge=1, le=5) for rating scales, as demonstrated in PhotoAnalysisOutput and ExperienceCreate models
- Always use Field(default_factory=dict) or Field(default_factory=list) for mutable collection fields to prevent shared mutable default bugs across instances
- For timestamp fields, use Field(default_factory=lambda: datetime.now(timezone.utc)) to ensure timezone-aware UTC timestamps, as shown in RSVPCreate, EventCreate, and AgentTask models
- Define nested context models (DiscordContext, TimerContext, EventContext) as separate BaseModel classes for reusability and clear validation boundaries, as demonstrated in agent state models

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' | grep -E '(models\.py|state\.py|agent_task\.py)' | wc -l
- grep -r 'Field(ge=' monorepo/tlt --include='*.py' | grep -E '(quality_score|relevance_score|vibe_score|confidence_score)' | wc -l
- grep -r 'default_factory=lambda: datetime.now(timezone.utc)' monorepo/tlt --include='*.py' | wc -l
- python -c 'from pydantic import BaseModel, Field; import ast; import sys; [print(f"PASS: {f}") for f in sys.argv[1:] if any("BaseModel" in n.name for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.Name))]' monorepo/tlt/mcp_services/*/models.py

Accept when:
- All model files in mcp_services/*/models.py, agents/*/state/*.py, and adapters/*/models.py define input models inheriting from Pydantic BaseModel
- Numeric fields with semantic constraints (scores, ratings, percentages) use Field validators with ge/le bounds
- All timestamp fields use timezone-aware datetime with default_factory for UTC timestamp generation
- Collection fields (Dict, List) consistently use Field(default_factory) to prevent mutable default bugs

## Enforcement

- Verified by: Code review checklist requiring Pydantic BaseModel for all new API models and external input handlers
- Verified by: CI pipeline static analysis using custom pylint rules to detect raw dict usage at API boundaries
- Verified by: Unit tests verifying validation behavior for boundary conditions (min/max values, required fields, type mismatches)
- Verified by: Integration tests submitting invalid payloads to API endpoints and verifying proper validation error responses
- Violation handling: CI pipeline fails if new models accepting external input do not inherit from BaseModel
- Violation handling: Code review blocks merge if validation constraints are missing for numeric fields with semantic bounds
- Violation handling: Runtime validation errors are logged with severity=ERROR and trigger alerting for repeated failures
- Violation handling: Security review required for any code bypassing validation at trust boundaries
- Exception process: Submit exception request to architecture review board with performance benchmarks or technical justification
- Exception process: Security team reviews exception for security implications and approves alternative validation strategy
- Exception process: Document exception in ADR exceptions registry with approval date, justification, and compensating controls
- Exception process: Exceptions reviewed quarterly to determine if technical constraints have been resolved