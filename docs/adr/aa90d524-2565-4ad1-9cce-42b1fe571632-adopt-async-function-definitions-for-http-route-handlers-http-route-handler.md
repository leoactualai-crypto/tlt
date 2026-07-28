# Adopt Async Function Definitions for HTTP Route Handlers: Http Route Handler

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Multiple service endpoints across the monorepo (event_manager, rsvp, discord_adapter) expose HTTP APIs using FastAPI framework with async route handlers
- Services require non-blocking I/O patterns to handle concurrent requests efficiently, particularly for health checks, RSVP management, and experience tracking endpoints
- The codebase demonstrates consistent use of async/await syntax across all HTTP route definitions, including GET, POST, PUT, and DELETE operations
- Integration with external systems (Discord API, database operations) necessitates asynchronous execution to prevent blocking the event loop
- The pattern appears in both MCP services (event_manager, rsvp) and adapter services (discord_adapter), indicating a monorepo-wide architectural convention

## Problem Statement

Services exposing HTTP APIs need to handle multiple concurrent requests without blocking, particularly when integrating with external systems or performing I/O operations. Synchronous route handlers would block the event loop, degrading throughput and increasing latency under load. A consistent concurrency model across service boundaries is required to ensure predictable performance characteristics and maintainability.

## Decision

1. MUST: All HTTP route handler functions MUST be defined with async def syntax

## Policy Block

- MUST All HTTP route handler functions MUST be defined with async def syntax

In scope:
- All FastAPI route handlers in MCP services (event_manager, rsvp)
- All FastAPI route handlers in adapter services (discord_adapter)
- Health check, ping, and monitoring endpoints
- CRUD operations for RSVP, experience, and event management
- Redirect handlers and URL rewriting endpoints

Out of scope:
- Background tasks or worker processes not serving HTTP requests
- CLI tools or scripts that do not expose HTTP endpoints
- Synchronous utility functions called within async route handlers
- Third-party library code outside the monorepo

Exceptions:
- EXC-001: A route handler performs only pure computation with no I/O, and profiling demonstrates that async overhead degrades performance

## Rationale

- Evidence shows 5 files across multiple services consistently implementing async route handlers, indicating an established architectural pattern with 91.62% confidence
- FastAPI framework is designed for async/await patterns and provides optimal performance when route handlers are asynchronous, particularly for I/O-bound operations
- The pattern enables horizontal scalability by allowing a single process to handle multiple concurrent requests without thread-per-request overhead
- Consistency across service boundaries (MCP services and adapters) reduces cognitive load and makes the codebase more maintainable

## Consequences

Positive:
- Improved throughput and reduced latency under concurrent load by preventing event loop blocking
- Consistent concurrency model across all service boundaries simplifies reasoning about system behavior
- Better resource utilization through cooperative multitasking rather than thread-based concurrency
- Natural integration with async libraries for database access, HTTP clients, and message queues

Negative:
- Async/await syntax adds complexity for developers unfamiliar with asynchronous programming patterns
- Debugging async code can be more challenging due to non-linear execution flow and stack traces
- Mixing sync and async code requires careful management of event loops and can introduce subtle bugs
- Small performance overhead for simple handlers that perform no I/O operations

## Alternatives

- Use synchronous route handlers with thread-based concurrency (WSGI servers like Gunicorn with sync workers) (rejected)
  Rejected because: Thread-based concurrency has higher memory overhead per request and does not integrate well with async libraries used for Discord API and database operations. Evidence shows the codebase has already committed to async patterns.
  When valid: Valid for legacy systems or when all dependencies are synchronous and cannot be migrated
- Mixed approach with async handlers only for I/O-bound routes and sync handlers for CPU-bound routes (rejected)
  Rejected because: Creates inconsistency across the codebase and requires developers to make case-by-case decisions. Evidence shows uniform async adoption across all route types including simple redirects and health checks.
  When valid: Valid when profiling demonstrates significant performance degradation from async overhead in specific CPU-bound endpoints
- Use async route handlers with explicit executor delegation for blocking operations (deferred)
  Rejected because: Not rejected; this is a complementary pattern for handling unavoidable blocking operations within async handlers
  When valid: Valid when integrating with synchronous libraries that cannot be replaced with async equivalents

## Risks

- Accidental blocking calls within async handlers (e.g., synchronous database drivers, time.sleep) can degrade performance for all concurrent requests
  Mitigation: Implement linting rules to detect blocking calls in async functions; use async-compatible libraries (asyncpg, aiohttp, etc.); add monitoring for event loop blocking
  Owner: Engineering team
- Developers unfamiliar with async patterns may introduce race conditions or improper await usage
  Mitigation: Provide async/await training and code review guidelines; establish patterns for common operations; use type hints and static analysis tools
  Owner: Engineering team
- Third-party libraries without async support may require workarounds or force synchronous execution
  Mitigation: Evaluate library async support during dependency selection; use run_in_executor for unavoidable blocking operations; maintain a list of approved async-compatible libraries
  Owner: Architecture team

## Implementation Notes

- Use FastAPI's dependency injection system to manage async database connections and HTTP clients with proper lifecycle management
- Implement health check endpoints with async def even when they perform minimal I/O to maintain consistency (as seen in health.py and routes.py)
- For redirect handlers like mcp_redirect_fix, use async def with immediate return of RedirectResponse to follow the established pattern
- Configure logging with async-compatible handlers (loguru is used in evidence) to prevent blocking on log writes under high load

## Continuation Context


Verify commands:
- grep -r 'def.*router\.' --include='*.py' monorepo/tlt/ | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'
- grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' -A 1 monorepo/tlt/ | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'
- python -m pytest tests/ -k 'test_async' -v --tb=short

Accept when:
- All route handler functions decorated with @router.* or @app.* use async def syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers

## Enforcement

- Verified by: Pre-commit hooks running grep patterns to detect synchronous route handlers
- Verified by: CI pipeline static analysis using pylint or flake8 with async-specific rules
- Verified by: Code review checklist requiring verification of async/await usage in route handlers
- Verified by: Automated tests validating that all registered routes use async handlers
- Violation handling: CI build fails if synchronous route handlers are detected in service code
- Violation handling: Code review blocks merge until synchronous handlers are converted to async or exception is documented
- Violation handling: Runtime monitoring alerts on event loop blocking exceeding threshold (e.g., >50ms)
- Violation handling: Quarterly architecture review identifies and remediates any synchronous handlers that bypassed checks
- Exception process: Developer submits exception request with performance profiling data demonstrating async overhead impact
- Exception process: Architecture team reviews exception request and validates that no async alternative exists
- Exception process: Approved exceptions are documented in code with inline comments and recorded in architecture decision log
- Exception process: Exceptions are reviewed quarterly to determine if async alternatives have become available