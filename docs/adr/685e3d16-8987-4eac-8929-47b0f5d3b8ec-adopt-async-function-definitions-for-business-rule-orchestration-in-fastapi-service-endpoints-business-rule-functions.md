# Adopt Async Function Definitions for Business Rule Orchestration in FastAPI Service Endpoints: Business Rule Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all FastAPI service endpoints that orchestrate business rules, handle external I/O operations, or coordinate Discord bot interactions within the TLT monorepo.

## Context

- The TLT monorepo contains multiple FastAPI-based services (reminder, experience_manager, rsvp, monitor) and MCP services (photo_processor) that orchestrate business rules through async function definitions
- All detected service endpoints use async/await patterns with asyncio for coordinating Discord bot operations, HTTP requests, database operations, and external API calls
- Business rule functions (create_reminder, schedule_reminder, handle_reaction, process_photo, health_check) are consistently defined as async functions decorated with FastAPI router methods
- The codebase integrates Discord.py async APIs (channel.fetch_message, thread.send, discord.utils.get) requiring async coordination within business rule handlers
- Pydantic BaseModel validation classes (ReminderCreate, ExperienceResponse, ReactionUpdate, TaskStatusResponse) are used synchronously for input validation before async business rule execution

## Problem Statement

FastAPI service endpoints that encode business rules must coordinate multiple I/O-bound operations including Discord API calls, HTTP requests, database queries, and external service interactions. Synchronous execution would block the event loop and degrade service responsiveness under concurrent load. The architecture requires a consistent concurrency model that enables non-blocking orchestration of business rule workflows while maintaining type safety and validation guarantees.

## Decision

1. MUST: Business rule functions that perform I/O operations (Discord API calls, HTTP requests, database queries) MUST use await for all async operations

## Policy Block

- MUST Business rule functions that perform I/O operations (Discord API calls, HTTP requests, database queries) MUST use await for all async operations

In scope:
- FastAPI router endpoint handlers in discord_adapter modules (reminder.py, experience_manager.py, rsvp.py)
- MCP service business rule processors (photo_processor.py)
- Service monitoring and health check endpoints (monitor.py)
- Any function that coordinates Discord bot operations, HTTP requests, or external API calls
- Business rule orchestration functions that manage state in bot.active_events, bot.active_reminders, or similar in-memory stores

Out of scope:
- Pydantic BaseModel class definitions for validation schemas
- Synchronous utility functions that perform pure computation without I/O
- Configuration loading and initialization code executed at module import time
- Type definitions, enums, and constant declarations
- Test fixtures and mock objects that simulate async behavior

Exceptions:
- EXC-001: A business rule function performs only CPU-bound computation with no I/O operations and execution time is guaranteed under 10ms
- EXC-002: Integration with a third-party library that only provides synchronous APIs and cannot be wrapped in asyncio.to_thread due to thread-safety constraints

## Rationale

- The evidence shows 5 files with 92.22% confidence consistently using async function definitions for business rule orchestration, indicating an established architectural pattern
- Discord.py async APIs (channel.fetch_message, thread.send) require await coordination, making async function definitions necessary for integration
- FastAPI natively supports async endpoint handlers and provides superior performance for I/O-bound workloads through non-blocking execution
- The pattern separates synchronous validation (Pydantic models) from async orchestration (business rule functions), providing clear boundaries between data validation and I/O coordination

## Consequences

Positive:
- Non-blocking I/O enables high concurrency and improved service responsiveness under load
- Consistent async/await patterns across all business rule handlers improve code readability and maintainability
- FastAPI async support provides automatic OpenAPI documentation and type checking for async endpoints
- Integration with Discord.py async APIs is natural and idiomatic without blocking the event loop

Negative:
- Async function definitions increase cognitive complexity for developers unfamiliar with asyncio programming model
- Debugging async code requires understanding of event loop mechanics and coroutine execution order
- Mixing async and sync code requires careful use of asyncio.to_thread or run_in_executor for blocking operations
- Testing async functions requires async test frameworks (pytest-asyncio) and understanding of event loop lifecycle

## Alternatives

- Use synchronous function definitions with blocking I/O and thread-based concurrency (rejected)
  Rejected because: Discord.py async APIs cannot be called from synchronous contexts without complex event loop management, and blocking I/O would degrade FastAPI performance under concurrent load
  When valid: Only valid for services with no Discord integration and guaranteed low concurrency requirements (< 10 concurrent requests)
- Use Celery task queue for all business rule orchestration with synchronous worker functions (rejected)
  Rejected because: Adds operational complexity (Redis/RabbitMQ dependency), increases latency for synchronous request-response patterns, and complicates error handling for user-facing endpoints
  When valid: Valid for long-running background tasks that do not require immediate response (e.g., batch photo processing, scheduled reminder execution)
- Use async function definitions only for Discord API calls, synchronous functions for other business rules (rejected)
  Rejected because: Creates inconsistent concurrency model across the codebase, requires complex bridging between async and sync contexts, and loses FastAPI async performance benefits
  When valid: Not recommended; consistency across business rule handlers is architecturally superior

## Risks

- Developers may introduce blocking I/O operations (synchronous requests.get, file I/O) within async functions, degrading event loop performance
  Mitigation: Implement linting rules to detect blocking calls in async functions, provide asyncio training, use aiohttp for HTTP requests and aiofiles for file I/O
  Owner: Engineering team with async/await expertise
- Unhandled exceptions in async business rule functions may cause event loop crashes or silent failures
  Mitigation: Wrap all async endpoint handlers with try-except blocks, use FastAPI exception handlers, implement structured logging for async execution traces
  Owner: Platform team responsible for FastAPI service infrastructure
- Testing async business rules requires proper event loop management and may introduce flaky tests if not handled correctly
  Mitigation: Standardize on pytest-asyncio with auto mode, provide async test fixtures, document async testing patterns in developer guide
  Owner: QA and testing infrastructure team

## Implementation Notes

- Use @router.post, @router.get, @router.delete decorators with async def function definitions for all FastAPI endpoints
- Apply Pydantic BaseModel validation through response_model parameter and function parameter type hints before async execution
- Use await for all Discord API calls (channel.fetch_message, thread.send, discord.utils.get), HTTP requests, and database queries
- Implement structured logging with logger.info, logger.error at key async execution points for observability
- Handle HTTPException within async functions to provide proper error responses for business rule violations

## Continuation Context


Verify commands:
- grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ monorepo/tlt/mcp_services/ | grep -E '(create_|handle_|process_|get_|delete_)' | wc -l
- grep -r '@router\.(post|get|delete|put)' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ | grep -v 'async def' && echo 'Found synchronous router handlers' || echo 'All router handlers are async'
- grep -r 'requests\.get\|requests\.post' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/mcp_services/ | grep -v 'timeout=' && echo 'Found blocking requests without timeout' || echo 'All requests have timeout or use async client'

Accept when:
- All FastAPI router endpoint handlers in discord_adapter and service modules are defined with async def syntax
- No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are present in async function bodies
- All Discord API calls use await and are coordinated within async function contexts
- Pydantic BaseModel validation is applied before async business rule execution in all endpoints

## Enforcement

- Verified by: CI pipeline linting with ruff or pylint configured to detect blocking calls in async functions
- Verified by: Code review checklist requiring verification of async/await patterns in business rule handlers
- Verified by: Automated grep-based verification in pre-commit hooks checking for synchronous router handlers
- Violation handling: CI build fails if synchronous router handlers are detected in FastAPI service modules
- Violation handling: Code review blocks merge if blocking I/O operations are found in async function bodies without justification
- Violation handling: Runtime monitoring alerts if event loop blocking is detected (execution time > 100ms for single async operation)
- Exception process: Developer submits exception request with performance benchmark or third-party library constraint evidence
- Exception process: Tech lead reviews exception request and evaluates async alternatives (asyncio.to_thread, aiohttp, aiofiles)
- Exception process: Approved exceptions are documented in function docstring with rationale and mitigation strategy
- Exception process: Exception registry is maintained in architecture decision log with periodic review