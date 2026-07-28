# Standardize Loguru for Structured Logging with Pydantic Domain Validation: Endpoints Agent Nodes

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Python services and agents within the monorepo that implement logging and domain validation patterns.

## Context

- The codebase demonstrates consistent pairing of loguru logger imports with Pydantic BaseModel validation across 8 files spanning services, adapters, agents, and MCP services
- Services require structured logging for operational visibility into task processing, event handling, health checks, and agent reasoning workflows
- Domain validation through Pydantic models provides type-safe contracts for API responses, task submissions, event creation, and agent decisions
- The pattern emerges in components handling external integrations (Discord adapter, photo processing), internal orchestration (event manager, monitor), and agent reasoning nodes
- Logging and validation co-occur at architectural boundaries where data enters or exits the system, requiring both observability and correctness guarantees

## Problem Statement

Services and agents require consistent operational observability while maintaining type-safe domain boundaries, but ad-hoc logging approaches and inconsistent validation patterns create gaps in debugging capability, increase error surface area at integration points, and complicate compliance verification across distributed components.

## Decision

1. MUST: All API endpoints, agent nodes, and external integration points MUST define domain models using Pydantic BaseModel with Field validators for input/output contracts

## Policy Block

- MUST All API endpoints, agent nodes, and external integration points MUST define domain models using Pydantic BaseModel with Field validators for input/output contracts

In scope:
- All Python modules in monorepo/tlt/services/
- All Python modules in monorepo/tlt/agents/
- All Python modules in monorepo/tlt/adapters/
- All Python modules in monorepo/tlt/mcp_services/
- FastAPI router definitions and endpoint handlers
- Agent reasoning nodes and workflow state machines
- External integration clients (Discord, HTTP APIs)

Out of scope:
- Third-party library code vendored in dependencies
- Test fixtures and mock objects that simulate logging
- Configuration files and deployment manifests
- Shell scripts and infrastructure automation

Exceptions:
- EXC-001: Legacy Discord adapter modules (event.py, reminder.py, experience_manager.py, rsvp.py) use Python standard logging.getLogger(__name__) for backward compatibility

## Rationale

- Evidence shows loguru adoption in 5 of 8 files (photo_processor.py, event_manager_clean.py, reasoning.py, monitor.py) while 3 Discord adapter files use standard logging, indicating active migration pattern
- Pydantic BaseModel validation appears consistently across all 8 files with Field constraints for scores (ge=0.0, le=1.0), demonstrating established domain validation practice
- Co-location of logging and validation at API boundaries (FastAPI routers), agent decision points (ReasoningNode), and health checks indicates architectural pattern for observable correctness
- Pattern support count of 8 files with 92.09% confidence across services, agents, and adapters demonstrates cross-cutting architectural concern rather than isolated implementation

## Consequences

Positive:
- Unified logging framework reduces cognitive overhead and enables consistent log aggregation across distributed services
- Type-safe domain models catch validation errors at API boundaries before propagating to business logic or persistence layers
- Structured logging with correlation IDs enables distributed tracing and root cause analysis across agent workflows and service interactions
- Health check logging provides operational visibility into service degradation and queue saturation conditions

Negative:
- Loguru dependency introduces additional runtime overhead compared to standard library logging, though impact is minimal for I/O-bound services
- Pydantic validation adds serialization/deserialization cost at API boundaries, increasing latency for high-throughput endpoints
- Mixed logging frameworks during migration period (loguru vs standard logging) complicate log aggregation and correlation
- Field constraint validation failures may expose internal model structure in error responses if not properly handled

## Alternatives

- Use Python standard library logging with structured formatters (JSON) across all modules (rejected)
  Rejected because: Standard logging requires verbose configuration for structured output and lacks loguru's ergonomic API for context binding and exception formatting
  When valid: When minimizing external dependencies is critical or when integrating with legacy systems that require standard logging handlers
- Use dataclasses with manual validation instead of Pydantic for domain models (rejected)
  Rejected because: Manual validation increases error surface area and lacks automatic OpenAPI schema generation for FastAPI endpoints
  When valid: For internal data structures that never cross API boundaries and do not require serialization
- Adopt OpenTelemetry for structured logging and tracing instead of loguru (deferred)
  Rejected because: OpenTelemetry provides superior distributed tracing but requires infrastructure investment in collectors and backends not yet available
  When valid: When distributed tracing infrastructure is deployed and cross-service correlation becomes critical operational requirement

## Risks

- Incomplete migration from standard logging to loguru creates inconsistent log formats and breaks log aggregation pipelines
  Mitigation: Establish migration timeline for Discord adapter modules, implement log format normalization in aggregation layer, add linting rules to detect new standard logging usage
  Owner: Platform Engineering Team
- Pydantic validation errors expose internal model structure or sensitive field names in API error responses
  Mitigation: Implement FastAPI exception handlers to sanitize Pydantic ValidationError responses, audit error messages for information disclosure
  Owner: Security Team
- Loguru's global logger configuration may conflict with third-party libraries that configure standard logging handlers
  Mitigation: Use loguru.logger.configure() to intercept standard logging, test integration with all third-party dependencies that emit logs
  Owner: Engineering Team

## Implementation Notes

- Import loguru logger at module level: 'from loguru import logger' - avoid lazy imports or conditional logger initialization
- Define Pydantic models with explicit Field validators for all numeric ranges, string patterns, and optional fields - use ge/le for bounds, description for API docs
- Emit logger.error() with exception context in all except blocks: logger.error(f'Operation failed: {e}') - include task_id or correlation ID when available
- Configure loguru sinks in service main() or __init__.py to route logs to stdout (JSON format) for container log aggregation
- Use Pydantic model_validate() for parsing untrusted input, Config.extra='forbid' to reject unknown fields at API boundaries

## Continuation Context


Verify commands:
- grep -r 'from loguru import logger' monorepo/tlt/services/ monorepo/tlt/agents/ monorepo/tlt/mcp_services/ | wc -l
- grep -r 'class.*BaseModel' monorepo/tlt/ | grep -v test | wc -l
- ruff check --select=G --select=LOG monorepo/tlt/ # Check for logging anti-patterns
- pytest tests/ -k 'test_validation' --cov=tlt --cov-report=term-missing

Accept when:
- All new Python modules in services/, agents/, adapters/, and mcp_services/ import loguru logger and define at least one logger statement
- All FastAPI endpoint handlers define Pydantic request/response models with Field validators for constrained parameters
- Health check endpoints emit logger statements for status transitions and include timestamp in ISO 8601 format
- Linting passes with no violations of logging anti-patterns (G*/LOG* rules) and Pydantic validation coverage exceeds 80%

## Enforcement

- Verified by: Pre-commit hooks run ruff linter to detect standard logging imports in new code outside exception list
- Verified by: CI pipeline runs pytest with coverage requirements for Pydantic model validation tests
- Verified by: Code review checklist includes verification of loguru usage and Pydantic Field constraints at API boundaries
- Verified by: Static analysis tools (mypy) enforce Pydantic model type annotations and detect missing Field validators
- Violation handling: CI build fails if new standard logging usage detected outside Discord adapter exception list
- Violation handling: Pull requests blocked if Pydantic models lack Field validators for numeric constraints or optional fields
- Violation handling: Runtime validation errors logged and reported to monitoring dashboard for pattern analysis
- Violation handling: Quarterly architecture review audits logging consistency and validation coverage across services
- Exception process: Submit exception request to architecture review board with justification for standard logging or unvalidated models
- Exception process: Document exception in module docstring with EXC-ID reference and migration timeline if temporary
- Exception process: Exceptions automatically expire after 6 months unless renewed with updated justification
- Exception process: All exceptions tracked in architecture decision log with approval date and owner