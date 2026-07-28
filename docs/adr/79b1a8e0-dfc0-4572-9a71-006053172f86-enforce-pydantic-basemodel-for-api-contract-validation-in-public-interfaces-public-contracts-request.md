# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Public Contracts Request

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all public API contracts, service boundaries, and domain validation models across the codebase.

## Context

- The codebase contains multiple MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager) and agents (ambient_event_agent) that expose public APIs and process external input from Discord, CloudEvents, and HTTP requests.
- Input validation failures in event-driven and microservice architectures can lead to runtime errors, data corruption, security vulnerabilities, and cascading failures across service boundaries.
- Pydantic BaseModel is already adopted across 11 files with 91.74% consistency, providing runtime type checking, field validation with constraints (ge, le, Field), and automatic serialization/deserialization.
- The pattern appears in critical security boundaries including user submissions (PhotoSubmission, RSVPCreate), authentication contexts (AuthContext, RBACRule), and inter-service contracts (MCPRequest, MCPResponse, AgentTask).
- Without enforced validation at API boundaries, services are vulnerable to type confusion, injection attacks, constraint violations, and malformed data propagation through the system.

## Problem Statement

Public API contracts and domain models lack consistent, enforceable input validation at service boundaries, creating security vulnerabilities and runtime stability risks. Without standardized validation, services must implement ad-hoc checks, leading to inconsistent error handling, incomplete constraint enforcement, and potential exploitation through malformed inputs. The system requires a uniform approach to validate data types, enforce business constraints, and sanitize inputs at all external interfaces.

## Decision

1. MUST: All public API contracts, request/response models, and domain entities MUST inherit from Pydantic BaseModel to enforce runtime type validation and constraint checking.

## Policy Block

- MUST All public API contracts, request/response models, and domain entities MUST inherit from Pydantic BaseModel to enforce runtime type validation and constraint checking.

In scope:
- All MCP service models (gateway, rsvp, event_manager, photo_vibe_check, guild_manager)
- Agent task models and state management (ambient_event_agent)
- Discord adapter experience and event models
- Authentication and authorization contexts (AuthContext, RBACRule)
- External API request/response contracts (MCPRequest, MCPResponse, RSVPCreate, EventCreate)
- Domain entities with business constraints (PhotoAnalysis, EventContext, AgentTask)

Out of scope:
- Internal data transfer objects used only within a single module with no external exposure
- Database ORM models that have separate validation layers
- Configuration objects loaded from trusted internal sources (not user input)
- Test fixtures and mock objects used exclusively in test suites

Exceptions:
- EXC-001: Performance-critical hot paths where validation overhead is measured and documented as unacceptable (>10ms p99 latency impact)
- EXC-002: Legacy integration points with third-party systems that cannot be modified and require custom parsing logic

## Rationale

- Evidence shows Pydantic BaseModel is consistently used across 11 files spanning MCP services, agents, and adapters, demonstrating established organizational practice with 91.74% adoption confidence.
- The pattern appears at critical security boundaries including user input (PhotoSubmission, RSVPCreate), authentication (AuthContext), and inter-service communication (MCPRequest, AgentTask), indicating its role in defense-in-depth strategy.
- Field-level constraints (ge=0.0, le=1.0 for scores, Field validation for required fields) are systematically applied in domain models, showing constraint enforcement is a deliberate architectural choice.
- The codebase processes untrusted input from multiple sources (Discord users, HTTP APIs, CloudEvents) requiring consistent validation to prevent injection attacks, type confusion, and constraint violations.

## Consequences

Positive:
- Runtime type safety and automatic validation catch malformed inputs before they reach business logic, preventing type confusion bugs and security vulnerabilities.
- Consistent error messages and validation failures across all services improve debugging and provide clear feedback to API consumers.
- Automatic JSON serialization/deserialization reduces boilerplate code and eliminates manual parsing errors.
- Field constraints (ge, le, regex) enforce business rules at the schema level, making validation logic explicit and self-documenting.
- IDE autocomplete and type checking work correctly with Pydantic models, improving developer productivity and reducing runtime errors.

Negative:
- Pydantic validation adds runtime overhead (typically 1-5ms per model instantiation) which may impact high-throughput endpoints processing thousands of requests per second.
- Complex nested models with deep validation can increase memory usage and CPU time during deserialization of large payloads.
- Pydantic version upgrades may introduce breaking changes requiring model updates across the codebase.
- Overly strict validation can reject valid edge cases, requiring careful design of Field constraints and custom validators.

## Alternatives

- Use Python dataclasses with manual validation logic in each service (rejected)
  Rejected because: Dataclasses provide no runtime validation, requiring manual type checking and constraint enforcement in every service, leading to inconsistent validation logic and security gaps. Evidence shows Pydantic is already adopted with Field constraints across 11 files.
  When valid: Only for internal DTOs with no external exposure where type hints alone are sufficient
- Implement custom validation framework using decorators and metaclasses (rejected)
  Rejected because: Building a custom validation framework requires significant engineering effort, ongoing maintenance, and lacks the ecosystem support, documentation, and battle-testing that Pydantic provides. Evidence shows Pydantic is already the de facto standard in the codebase.
  When valid: Never - Pydantic provides all required functionality
- Use JSON Schema validation with jsonschema library (rejected)
  Rejected because: JSON Schema validation is decoupled from Python type system, provides no IDE support, requires separate schema definitions, and lacks automatic serialization. Pydantic generates JSON Schema automatically while providing Python-native validation.
  When valid: Only when integrating with external systems that require JSON Schema as the contract format

## Risks

- Performance degradation in high-throughput endpoints due to Pydantic validation overhead, particularly for complex nested models or large batch operations.
  Mitigation: Profile validation performance in critical paths, use Pydantic's parse_obj_as for batch operations, consider validation caching for repeated patterns, and document performance benchmarks for exception requests.
  Owner: Platform Engineering Team
- Incomplete validation coverage if developers bypass Pydantic models by accepting raw dictionaries or using type: ignore comments to suppress validation errors.
  Mitigation: Implement linting rules to detect raw dict usage at API boundaries, require code review for validation bypasses, add CI checks to verify all public endpoints use Pydantic models, and provide training on proper model usage.
  Owner: Security Team
- Breaking changes during Pydantic version upgrades (especially v1 to v2 migration) requiring extensive model refactoring across the codebase.
  Mitigation: Pin Pydantic version in requirements.txt, test upgrades in isolated environment, use Pydantic's migration guide, implement comprehensive validation test suite to catch breaking changes, and schedule dedicated migration sprints.
  Owner: Engineering Team

## Implementation Notes

- Use Field(default_factory=dict) for mutable defaults and Field(default_factory=lambda: datetime.now(timezone.utc)) for timestamps to prevent shared state bugs.
- Apply Field constraints (ge, le, min_length, max_length, regex) for all business rules and security boundaries - examples: quality_score: float = Field(ge=0.0, le=1.0), emoji: str = Field(regex=r'^\p{Emoji}$').
- Define explicit Config classes with json_encoders for custom types (datetime: lambda v: v.isoformat()) to ensure consistent serialization across services.
- Implement @validator decorators for complex validation logic that cannot be expressed through Field constraints, and use @root_validator for cross-field validation.
- Provide descriptive Field(description='...') annotations for all public API fields to support automatic OpenAPI documentation generation.
- Use Optional[T] for nullable fields and provide sensible defaults where appropriate, avoiding Optional for required security-critical fields like user_id or event_id.

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l
- grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'
- python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

Accept when:
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.

## Enforcement

- Verified by: Pre-commit hooks running pylint/mypy to detect missing type annotations and BaseModel inheritance at API boundaries.
- Verified by: CI pipeline checks using grep patterns to verify all models.py files contain BaseModel inheritance and Field constraints.
- Verified by: Code review checklist requiring validation model review for all new API endpoints and external input handlers.
- Verified by: Automated security scanning to detect raw dict usage in request handlers and flag for manual review.
- Violation handling: CI build fails if new API endpoints are added without Pydantic validation models.
- Violation handling: Security team review required for any validation bypass or type: ignore comments in API boundary code.
- Violation handling: Pull requests adding raw dict parameters to public methods are automatically flagged and require architecture approval.
- Violation handling: Quarterly security audits identify validation gaps and create remediation tickets with priority based on exposure risk.
- Exception process: Submit exception request to architecture review board with performance benchmarks or technical justification.
- Exception process: Security team reviews alternative validation approach and approves compensating controls.
- Exception process: Document exception in code comments with ADR reference, rationale, and monitoring strategy.
- Exception process: Exceptions are reviewed quarterly and must be re-justified or remediated within 6 months.