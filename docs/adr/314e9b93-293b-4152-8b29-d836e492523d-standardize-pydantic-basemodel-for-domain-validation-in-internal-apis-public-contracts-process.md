# Standardize Pydantic BaseModel for Domain Validation in Internal APIs: Public Contracts Process

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Internal API contracts across photo processing, event management, and experience tracking services require structured validation of domain models
- Multiple services independently adopted Pydantic BaseModel with Field constraints for input/output validation, creating a de facto standard
- Domain models include PhotoAnalysisOutput, GenZVibeCheckOutput, CloudEventContext, EventContext, DiscordContext, TimerContext, IncomingEvent, ExperienceCreate, and ExperienceResponse
- Services expose public contracts through process_photo, process_genz_vibe_check, create_experience, get_experience, and event lifecycle tracking functions
- Validation requirements span quality scores (0.0-1.0), categorical ratings, size checks, timestamps, user IDs, and nested context structures

## Problem Statement

Internal APIs require consistent validation of domain models to ensure data integrity across service boundaries, prevent invalid state propagation, and provide clear contract definitions for inter-service communication without imposing runtime overhead or coupling services to specific validation frameworks.

## Decision

1. MUST: Public API contracts (process_*, create_*, get_*) MUST accept and return validated BaseModel instances

## Policy Block

- MUST Public API contracts (process_*, create_*, get_*) MUST accept and return validated BaseModel instances

In scope:
- Internal API contracts between monorepo services
- Domain models for photo processing, event management, and experience tracking
- Input validation for external client requests (Discord, HTTP endpoints)
- Output validation for service responses
- State management structures (IncomingEvent, PhotoProcessingWorkflowState)

Out of scope:
- External API contracts with third-party services
- Database schema definitions
- Configuration file validation
- CLI argument parsing
- Logging message structures

## Rationale

- Pattern detected across 3 files with 92.30% confidence, demonstrating consistent adoption of Pydantic BaseModel for domain validation in internal APIs
- Pydantic provides declarative validation with minimal boilerplate, automatic type coercion, and clear error messages for contract violations
- Field constraints (ge, le, description) encode domain invariants directly in model definitions, making validation rules self-documenting and enforceable at runtime
- Decomposition into context-specific models (CloudEventContext, DiscordContext, TimerContext) separates concerns and enables reuse across different event trigger types

## Consequences

Positive:
- Validation errors are caught at API boundaries before invalid data propagates through the system
- Self-documenting contracts reduce cognitive load for developers integrating with internal APIs
- Type hints and IDE support improve developer experience and reduce runtime errors
- Consistent validation approach reduces maintenance burden and enables shared validation utilities

Negative:
- Pydantic introduces runtime overhead for validation on every model instantiation
- Tight coupling to Pydantic framework makes migration to alternative validation libraries costly
- Complex validation logic may require custom validators, reducing declarative clarity
- Nested models with deep validation hierarchies can produce verbose error messages

## Alternatives

- Use dataclasses with manual validation functions (rejected)
  Rejected because: Manual validation is error-prone, requires boilerplate for each model, and lacks automatic type coercion and constraint enforcement
  When valid: When validation logic is trivial or performance overhead of Pydantic is unacceptable
- Use marshmallow schemas for validation (rejected)
  Rejected because: Marshmallow separates schema from data classes, requiring duplicate definitions and lacking native type hint integration
  When valid: When serialization/deserialization flexibility is more important than type safety
- Use attrs with validators (rejected)
  Rejected because: Attrs provides less declarative constraint syntax and lacks Field-level metadata for documentation
  When valid: When immutability and slots optimization are primary concerns

## Risks

- Pydantic version upgrades may introduce breaking changes to validation behavior or Field API
  Mitigation: Pin Pydantic major version in requirements, test validation behavior in CI, review Pydantic changelog before upgrades
  Owner: engineering team
- Performance overhead of validation may impact high-throughput API endpoints
  Mitigation: Profile validation overhead in performance tests, use Pydantic's model_construct() for trusted internal data, consider validation caching
  Owner: engineering team
- Complex nested models may produce unclear validation errors for API consumers
  Mitigation: Implement custom error formatting, provide validation error examples in API documentation, use clear Field descriptions
  Owner: engineering team

## Implementation Notes

- Import BaseModel and Field from pydantic for all domain validation models
- Use Field(ge=0.0, le=1.0, description='...') for numeric constraints with documentation
- Use Field(default_factory=dict) or Field(default_factory=list) for mutable default values to avoid shared state bugs
- Decompose complex models into focused context classes (e.g., separate DiscordContext from EventContext) for reusability
- Include reasoning or metadata fields in output models to support debugging and observability
- Use Optional[T] type hints with default=None for optional fields
- Test validation behavior with pytest fixtures covering valid, invalid, and edge-case inputs

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Output|Context|Create|Response|Event)' | wc -l
- grep -r 'Field(ge=' --include='*.py' monorepo/tlt/ | wc -l
- python -c "import ast; import sys; files=['monorepo/tlt/mcp_services/photo_vibe_check/photo_processor.py', 'monorepo/tlt/agents/ambient_event_agent/state/state.py', 'monorepo/tlt/adapters/discord_adapter/experience_manager.py']; [ast.parse(open(f).read()) for f in files]; print('Validation models parse successfully')"

Accept when:
- At least 3 domain validation models inherit from Pydantic BaseModel across internal API services
- Numeric score fields use Field constraints (ge, le) to enforce valid ranges
- All Field definitions include description parameters for self-documentation
- Python AST parsing confirms validation models are syntactically valid

## Enforcement

- Verified by: Code review checks for BaseModel inheritance in new domain models
- Verified by: CI linting rules verify Field constraints on numeric score fields
- Verified by: Unit tests validate constraint enforcement (e.g., scores outside 0.0-1.0 raise ValidationError)
- Verified by: Type checking with mypy verifies Optional and type hint correctness
- Violation handling: Code review blocks merge if domain models lack BaseModel inheritance or Field constraints
- Violation handling: CI fails if validation tests do not cover constraint violations
- Violation handling: Runtime ValidationError exceptions are logged and returned as 400 Bad Request to API clients
- Violation handling: Quarterly audits identify validation models missing description fields
- Exception process: Request exception via architecture review for performance-critical paths where validation overhead is measured and unacceptable
- Exception process: Document exception rationale in ADR amendment with benchmark data
- Exception process: Exceptions require alternative validation strategy (e.g., manual checks, schema validation at gateway)
- Exception process: Re-evaluate exceptions after Pydantic performance improvements or migration to compiled validators