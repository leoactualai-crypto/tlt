# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Endpoint Handlers Encoding

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all services implementing business rules, domain validation, and API contracts within the monorepo/tlt domain.

## Context

- The monorepo/tlt codebase implements multiple FastAPI-based services (discord_adapter, tlt_service, mcp_services) that encode business rules for event management, reminders, experience tracking, photo processing, and monitoring.
- Each service module combines domain validation (Pydantic BaseModel schemas), API contracts (FastAPI routers), and business logic that requires operational visibility into rule execution, validation failures, and state transitions.
- The pattern emerged across 5 files with 92.22% confidence, showing consistent use of logging.getLogger(__name__) and loguru.logger alongside domain validation classes and API endpoint definitions.
- Business rules encoded in these services include RSVP reaction handling, reminder scheduling, experience rating validation, photo quality assessment, and health check status determination.
- The logging infrastructure provides runtime traceability for business rule execution, enabling debugging of validation failures, monitoring of rule violations, and audit trails for domain events.

## Problem Statement

Services encoding business rules through domain validation models and API contracts require structured logging to trace rule execution, diagnose validation failures, monitor compliance, and maintain audit trails. Without standardized logging practices tied to business rule modules, teams face difficulty correlating log entries with specific rule violations, debugging complex validation chains, and establishing operational observability for domain logic.

## Decision

1. SHOULD: API endpoint handlers encoding business rules SHOULD log request processing outcomes (success, validation failure, not found, server error) with correlation identifiers to support distributed tracing.

## Policy Block

- SHOULD API endpoint handlers encoding business rules SHOULD log request processing outcomes (success, validation failure, not found, server error) with correlation identifiers to support distributed tracing.

In scope:
- FastAPI router modules implementing business rule endpoints (reminder.py, experience_manager.py, rsvp.py, monitor.py)
- Domain validation modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse)
- Business logic processors implementing rule evaluation (photo_processor.py quality checks, monitor.py health status determination)
- Service modules coordinating async business workflows (reminder scheduling, RSVP reaction handling, experience statistics aggregation)

Out of scope:
- Pure data transfer objects without validation logic
- Static configuration files and schema definitions
- Test fixtures and mock implementations
- Third-party library integration code that does not encode domain-specific business rules

Exceptions:
- EXC-001: Performance-critical hot paths where logging overhead is measured to exceed 5% of execution time
- EXC-002: Ephemeral lambda functions or serverless handlers with external centralized logging infrastructure

## Rationale

- The evidence shows consistent co-occurrence of logging initialization (logging.getLogger(__name__), loguru.logger) with domain validation (BaseModel schemas) and API contracts (FastAPI routers) across 5 files with 92.22% significance, indicating an established pattern for business rule observability.
- Business rules encoded in these services require runtime traceability: RSVP reaction handling logs user actions, reminder scheduling logs creation/deletion, experience manager logs rating submissions, photo processor logs quality assessments, and monitor logs health state transitions.
- Module-scoped logger naming using __name__ preserves the architectural boundary structure (adapters/discord_adapter, services/tlt_service, mcp_services/photo_vibe_check), enabling log filtering aligned with domain boundaries and service ownership.
- Structured logging at business rule execution points provides the operational foundation for debugging validation failures (HTTPException with 404/500 status codes), monitoring rule compliance (health check status determination), and establishing audit trails for domain events (user reactions, reminder operations).

## Consequences

Positive:
- Enables correlation of log entries with specific business rule violations and validation failures through module-scoped logger names and contextual identifiers (user_id, message_id, event_id)
- Provides operational visibility into business rule execution patterns, supporting debugging of complex validation chains and monitoring of service health states
- Establishes audit trails for domain events (RSVP reactions, reminder operations, experience submissions) required for compliance and user support
- Facilitates log aggregation and filtering aligned with architectural boundaries (adapters, services, mcp_services) and service ownership

Negative:
- Introduces logging overhead in business rule execution paths, potentially impacting latency in high-throughput validation scenarios
- Requires discipline to maintain consistent logging practices across teams and services, with risk of log quality degradation without enforcement
- May generate high log volumes in services with frequent business rule evaluations (RSVP reactions, photo processing), requiring log retention and cost management
- Creates coupling between business logic and logging infrastructure, complicating testing and requiring log mocking in unit tests

## Alternatives

- Use unstructured print statements or ad-hoc logging without module-scoped loggers (rejected)
  Rejected because: Unstructured logging prevents log filtering by module/domain, lacks severity levels for operational alerting, and does not support log aggregation infrastructure. Evidence shows consistent use of structured logging frameworks (logging.getLogger, loguru) across all business rule modules.
  When valid: Never valid for production business rule encoding; acceptable only for temporary debugging in local development
- Rely solely on metrics and tracing without detailed logging for business rule execution (rejected)
  Rejected because: Metrics provide aggregates but lack the contextual detail (user_id, validation failure reasons, input data) required to debug specific business rule violations. Evidence shows logging used to capture detailed context (logger.info with user names, event topics, reaction emojis) not available in metrics.
  When valid: Valid as complementary observability for performance monitoring, but insufficient alone for business rule traceability
- Centralize all business rule logging in a dedicated observability service with structured event emission (deferred)
  Rejected because: Would require significant refactoring of existing services and introduction of event emission infrastructure. Current pattern of module-scoped logging is established and functional.
  When valid: Consider for future architecture evolution if cross-service business rule correlation becomes a primary requirement or log volumes require specialized handling

## Risks

- Inconsistent logging practices across services lead to gaps in business rule observability, with some modules logging validation failures while others silently fail
  Mitigation: Establish linting rules to detect missing logger initialization in modules with BaseModel or FastAPI router definitions. Implement code review checklist requiring logging for business rule execution paths. Provide logging templates and examples in developer documentation.
  Owner: Platform Engineering Team
- High log volumes from frequent business rule evaluations (RSVP reactions, photo processing) exceed log retention budgets or impact service performance
  Mitigation: Implement log sampling for high-frequency operations while preserving full logging for validation failures and errors. Configure log levels per environment (DEBUG in dev, INFO in staging, WARNING in production for non-critical paths). Monitor logging overhead and establish performance budgets.
  Owner: SRE Team
- Sensitive data (user feedback, photo content analysis) inadvertently logged in business rule execution traces, creating compliance exposure
  Mitigation: Establish logging sanitization guidelines prohibiting PII and sensitive content in log messages. Use structured logging with explicit field inclusion rather than logging entire request/response objects. Implement automated scanning for sensitive data patterns in logs.
  Owner: Security Team

## Implementation Notes

- Initialize module-scoped loggers at the top of each business rule module: `logger = logging.getLogger(__name__)` for standard library or `from loguru import logger` for loguru-based services. Preserve module naming to maintain architectural boundary visibility.
- Log business rule execution outcomes with contextual identifiers: `logger.info(f'User {user.name} reacted with {emoji} to event: {event["topic"]}')` for successful operations, `logger.error(f'Error handling reaction: {e}')` for failures. Include entity IDs (user_id, message_id, event_id) for correlation.
- For health check and monitoring endpoints, log status transitions with structured context: `logger.error(f'Health check failed: {e}')` with timestamp and service name. Use consistent status values (healthy, degraded, unhealthy, warning) for operational alerting.
- Configure log levels per environment and module: DEBUG for development, INFO for staging, WARNING/ERROR for production business rule paths. Use log sampling for high-frequency operations (reactions, photo processing) while preserving full logging for validation failures.

## Continuation Context


Verify commands:
- grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'
- grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'
- grep -r '@router\.(get\|post\|delete)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l

Accept when:
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using logging.getLogger(__name__) or loguru.logger
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)

## Enforcement

- Verified by: Static analysis linting rules detecting missing logger initialization in modules with BaseModel or FastAPI router definitions
- Verified by: Code review checklist requiring logging verification for business rule execution paths and validation failure handling
- Verified by: Automated grep-based verification in CI pipeline checking for logger presence in business rule modules
- Violation handling: CI pipeline fails if business rule modules lack logger initialization (detected via static analysis)
- Violation handling: Code review blocks merge if validation failure paths do not emit structured log entries with contextual identifiers
- Violation handling: Quarterly audit of logging practices with remediation tickets for modules missing business rule traceability
- Exception process: Submit exception request to Architecture Review Board with performance profiling evidence (for performance-critical hot paths) or external logging infrastructure documentation (for serverless handlers)
- Exception process: Document approved exceptions in module docstring with rationale, alternative observability mechanism, and approval reference
- Exception process: Review exceptions annually to assess if constraints still apply or if logging can be reintroduced