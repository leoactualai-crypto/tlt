# Standardize Logging with Named Loggers and Pydantic Validation Models: Endpoints That Accept

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple service adapters (Discord adapter modules) and services (tlt_service) that require structured observability for debugging and operational monitoring
- Logging is consistently paired with Pydantic BaseModel validation classes across API endpoints, indicating a pattern of validating inputs before logging operations
- Both standard library logging (logging.getLogger) and third-party loguru (from loguru import logger) are used across different modules, suggesting an evolving logging strategy
- FastAPI router endpoints consistently implement logging alongside request/response validation models for API contracts
- The pattern appears in 5 files with 92.22% confidence, spanning Discord adapters (reminder.py, experience_manager.py, rsvp.py), MCP services (photo_processor.py), and core services (monitor.py)

## Problem Statement

Services and adapters require consistent logging practices to enable debugging, operational monitoring, and audit trails, while simultaneously ensuring that logged data is validated and structured through domain models to prevent logging of malformed or unsafe data.

## Decision

1. MUST: API endpoints that accept external input MUST define Pydantic BaseModel validation classes for request and response contracts

## Policy Block

- MUST API endpoints that accept external input MUST define Pydantic BaseModel validation classes for request and response contracts

In scope:
- FastAPI router endpoints accepting external requests
- Discord adapter modules handling bot events and commands
- MCP service modules processing external data (photos, files)
- Service monitoring and health check endpoints
- Any module that performs I/O operations or external API calls

Out of scope:
- Pure utility functions with no I/O or external dependencies
- Internal data transformation functions that operate on already-validated data
- Test fixtures and mock implementations
- Configuration loading modules that use environment variables only

Exceptions:
- EXC-001: Legacy modules undergoing gradual migration may temporarily use print() statements
- EXC-002: Performance-critical hot paths may defer validation to reduce latency

## Rationale

- The evidence shows consistent co-occurrence of logging initialization (logging.getLogger, loguru logger) with Pydantic BaseModel validation classes across 5 files, indicating an established pattern of pairing observability with input validation
- Named loggers using __name__ enable hierarchical configuration and filtering by module, improving operational debugging capabilities across the Discord adapter and service layers
- Validating inputs before logging prevents injection of malformed data into logs and ensures structured, queryable log output
- The pattern spans multiple architectural boundaries (adapters, services, MCP processors) with 92.22% confidence, suggesting this is an intentional architectural decision rather than coincidental implementation

## Consequences

Positive:
- Structured logging with named loggers enables filtering and aggregation by module, improving debugging efficiency
- Pydantic validation ensures type safety and constraint enforcement before data reaches business logic or logs
- Consistent logging patterns across adapters and services reduce cognitive load for developers navigating the codebase
- Validated log data enables reliable log parsing and analysis in monitoring systems

Negative:
- Dual logging libraries (logging and loguru) increase dependency footprint and require developers to know which library is used in each module
- Validation overhead adds latency to request processing, particularly for high-frequency endpoints
- Pydantic model definitions increase code volume and maintenance burden for simple data structures
- Logger initialization boilerplate must be repeated in every module

## Alternatives

- Use unvalidated dictionaries for API contracts and log all inputs without validation (rejected)
  Rejected because: Eliminates type safety and allows malformed data to propagate into logs and business logic, increasing debugging difficulty and security risk
  When valid: Never valid for production code; acceptable only in throwaway prototypes
- Standardize on a single logging library (either logging or loguru) across all modules (deferred)
  Rejected because: Not rejected; evidence shows mixed usage but no clear migration path documented
  When valid: Valid as a future refactoring initiative once feature requirements for both libraries are assessed
- Use OpenTelemetry structured logging with automatic context propagation (rejected)
  Rejected because: No evidence of OpenTelemetry usage in the codebase; would require significant infrastructure investment and migration effort
  When valid: Valid if distributed tracing and cross-service correlation become requirements

## Risks

- Inconsistent logging library usage (logging vs loguru) creates confusion and prevents unified log configuration
  Mitigation: Document library selection criteria and create migration plan to standardize on one library
  Owner: Platform engineering team
- Validation overhead in high-frequency endpoints may cause latency spikes or throughput degradation
  Mitigation: Profile validation performance and implement caching or lazy validation for hot paths
  Owner: Service owners
- Developers may bypass validation by logging before validation completes, undermining data integrity
  Mitigation: Implement linting rules to detect logging calls before validation and enforce in code review
  Owner: Engineering team

## Implementation Notes

- Initialize loggers at module level using `logger = logging.getLogger(__name__)` or `from loguru import logger` immediately after imports
- Define Pydantic BaseModel classes for all FastAPI router request and response types, using Field() for constraints and documentation
- Structure endpoint handlers to validate inputs first (via FastAPI dependency injection), then log operations, then execute business logic
- Use logger.info() for successful operations, logger.error() for failures, and include contextual data (user_id, message_id, etc.) in log messages
- For modules using standard logging, configure handlers and formatters in the service entry point (main.py) to ensure consistent output format

## Continuation Context


Verify commands:
- grep -r 'logging.getLogger(__name__)' --include='*.py' monorepo/tlt/
- grep -r 'from loguru import logger' --include='*.py' monorepo/tlt/
- grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -E '(Request|Response|Create|Update|Output)'
- python -m pytest tests/ -k 'validation' -v

Accept when:
- All modules with I/O operations or external API calls initialize a named logger
- All FastAPI router endpoints define Pydantic BaseModel validation classes for request/response contracts
- Logging statements occur after validation succeeds, verified by code review or static analysis
- No print() statements exist in production code paths (excluding explicitly documented exceptions)

## Enforcement

- Verified by: Code review checklist requiring logger initialization and validation model presence
- Verified by: Static analysis with grep or AST-based linting to detect missing loggers or validation models
- Verified by: CI pipeline checks for print() statements in non-test code
- Verified by: Manual inspection during architecture review for new services or adapters
- Violation handling: Pull requests missing logger initialization or validation models are blocked until corrected
- Violation handling: Existing violations are tracked in technical debt backlog with priority based on module criticality
- Violation handling: Print statements in production code trigger CI failure and require immediate remediation
- Violation handling: Repeated violations by a team trigger architecture review and additional training
- Exception process: Developer submits exception request to tech lead with justification (performance, legacy migration, etc.)
- Exception process: Tech lead reviews with architecture team if exception impacts multiple modules or services
- Exception process: Approved exceptions are documented in module docstring with expiration date or migration plan
- Exception process: Exception registry is reviewed quarterly to ensure temporary exceptions do not become permanent