# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Components Performing Real

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all Discord adapter components that perform real-time message operations and require operational observability.

## Context

- Discord adapter components (reminder.py, bot_manager.py) perform real-time asynchronous operations including message sending, thread creation, and reminder scheduling that require operational visibility
- The codebase uses Python's standard logging library with module-level named loggers via logging.getLogger(__name__) to provide structured observability across async Discord operations
- Real-time boundaries involve await-based message sending (thread.send, channel.send, user.send) and external HTTP client calls to TLT service endpoints that need traceable execution paths
- Integration with FastAPI routers, Discord bot lifecycle hooks (on_ready, on_guild_join, on_reaction_add), and scheduled tasks (reminder_check_task) creates multiple concurrent execution contexts requiring coordinated logging
- The pattern emerged across 2 files with 92.20% confidence, indicating consistent adoption of standard logging for observability in real-time Discord integration boundaries

## Problem Statement

Real-time Discord operations involving asynchronous message delivery, HTTP client interactions, and scheduled task execution require consistent operational observability to diagnose failures, trace execution flows, and monitor system health across multiple concurrent contexts without introducing custom logging abstractions that increase maintenance burden.

## Decision

1. MUST: Components performing real-time operations (await thread.send, channel.send, user.send) MUST log operation initiation and completion status

## Policy Block

- MUST Components performing real-time operations (await thread.send, channel.send, user.send) MUST log operation initiation and completion status

In scope:
- Discord adapter modules (reminder.py, bot_manager.py)
- Real-time message operations using Discord.py async methods (send, thread creation)
- HTTP client interactions with TLT service endpoints
- Bot lifecycle event handlers and scheduled background tasks
- FastAPI router endpoints managing Discord-related resources

Out of scope:
- Non-Discord adapter components in other service layers
- Synchronous operations without real-time boundaries
- Third-party library internal logging (Discord.py, aiohttp, FastAPI internal logs)
- Application-level metrics collection or tracing systems

Exceptions:
- EXC-001: Performance-critical hot paths where logging overhead is measured to exceed 5% of operation latency

## Rationale

- Python's standard logging library provides mature, well-understood observability without additional dependencies, reducing maintenance burden and integration complexity
- Named loggers via getLogger(__name__) enable hierarchical logger configuration and filtering, allowing operators to adjust verbosity per module without code changes
- Real-time Discord operations span multiple async contexts (message sending, HTTP calls, scheduled tasks) where consistent logging provides essential execution traceability for debugging failures
- Evidence shows 92.20% confidence across 2 files with consistent adoption of logging.getLogger(__name__) pattern, indicating organic convergence on this approach for Discord adapter observability

## Consequences

Positive:
- Operators gain visibility into real-time Discord message delivery, reminder scheduling, and bot lifecycle events through standard logging infrastructure
- Module-scoped named loggers enable fine-grained log level control per component without code modification
- Standard logging integration allows existing log aggregation, filtering, and alerting tools to process Discord adapter logs without custom parsers
- Reduced maintenance burden by avoiding custom logging abstractions while maintaining observability across async execution contexts

Negative:
- Standard logging lacks structured logging features (JSON output, typed fields) without additional configuration or formatters
- High-frequency real-time operations may generate significant log volume requiring careful level configuration to avoid I/O bottlenecks
- Correlation across distributed async operations requires manual inclusion of context IDs (reminder_id, message_id) in log messages
- No built-in support for distributed tracing or span correlation without integrating additional observability libraries

## Alternatives

- Implement custom logging wrapper with structured logging and automatic context injection (rejected)
  Rejected because: Increases maintenance burden, introduces custom abstraction layer, and evidence shows standard logging meets current observability needs across 2 files with 92.20% confidence
  When valid: If distributed tracing or mandatory structured logging becomes a cross-cutting requirement with dedicated observability team support
- Adopt third-party structured logging library (structlog, loguru) for enhanced features (rejected)
  Rejected because: Adds external dependency, requires team training, and current evidence shows standard logging provides sufficient observability for real-time Discord operations
  When valid: If log volume analysis demonstrates need for structured filtering or if compliance requirements mandate structured audit logs
- Use print statements or Discord channel logging for operational visibility (rejected)
  Rejected because: Lacks configurability, filtering, level control, and integration with standard logging infrastructure; unsuitable for production observability
  When valid: Never valid for production code; acceptable only for temporary local debugging

## Risks

- High-frequency real-time message operations generate excessive log volume causing I/O bottlenecks or storage exhaustion
  Mitigation: Configure log levels appropriately (INFO for lifecycle events, DEBUG for detailed operations), implement log rotation, and monitor log volume metrics with alerting thresholds
  Owner: Engineering team with operations support
- Lack of structured logging makes automated log parsing and correlation difficult for incident response
  Mitigation: Establish log message format conventions including context IDs (reminder_id, message_id, guild_id), document format in team guidelines, and evaluate structured logging formatters if parsing becomes bottleneck
  Owner: Engineering team
- Inconsistent logging practices across modules reduce observability effectiveness
  Mitigation: Document logging standards in this ADR, include logging verification in code review checklist, and add linting rules to detect missing logger initialization
  Owner: Engineering team

## Implementation Notes

- Initialize module logger at top of each Discord adapter file: logger = logging.getLogger(__name__)
- Log real-time operation boundaries: logger.info before await send operations, logger.error for exceptions with context IDs
- Include relevant context in log messages: reminder_id for reminder operations, message_id/channel_id for Discord operations, guild_id for bot events
- Configure logging level via environment variable or config file to allow runtime adjustment without code changes
- Use logger.exception() in exception handlers to automatically capture stack traces for debugging

## Continuation Context


Verify commands:
- grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/
- grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l
- python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO

Accept when:
- All Discord adapter modules contain 'logging.getLogger(__name__)' initialization
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules

## Enforcement

- Verified by: Code review checklist verifying logger initialization and operation logging
- Verified by: Automated grep-based CI check for logging.getLogger(__name__) pattern in new Discord adapter modules
- Verified by: Integration test execution with log level validation
- Violation handling: Code review feedback requesting addition of standard logging before merge approval
- Violation handling: CI pipeline warning on missing logger initialization in Discord adapter modules
- Violation handling: Post-merge remediation tracked as technical debt ticket if violation reaches production
- Exception process: Document exception request with performance benchmark evidence showing logging overhead impact
- Exception process: Engineering lead reviews exception with alternative observability approach (metrics, tracing)
- Exception process: Approved exceptions documented in module docstring with monitoring alternative and review date