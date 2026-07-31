# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Discord Adapter Modules

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer.

## Context

- The TLT service integrates with Discord through adapter components that require asynchronous communication patterns to handle real-time message delivery, reminder scheduling, and event notifications
- Discord's API enforces async/await patterns for all I/O operations including channel.send(), thread.send(), and user.send() methods, requiring the adapter layer to adopt async concurrency models
- The system coordinates between HTTP-based REST endpoints (FastAPI routers) and Discord's event-driven bot lifecycle, necessitating boundary definitions for real-time vs request-response patterns
- Multiple adapter modules (reminder.py, bot_manager.py) independently implement async messaging boundaries, indicating a consistent architectural pattern across the integration layer
- The integration layer must maintain state synchronization between Discord events and the TLT service backend through periodic polling and event-driven updates

## Problem Statement

Integration boundaries between HTTP-based services and real-time messaging platforms like Discord require explicit architectural patterns to handle async I/O, event-driven workflows, and bidirectional communication without introducing blocking operations, race conditions, or state inconsistencies across the adapter layer.

## Decision

1. MUST: Discord adapter modules MUST implement async function signatures for all public API contracts including create_reminder, schedule_reminder, get_reminder, list_reminders, delete_reminder, setup_hook, on_ready, on_guild_join, on_guild_remove, and on_reaction_add

## Policy Block

- MUST Discord adapter modules MUST implement async function signatures for all public API contracts including create_reminder, schedule_reminder, get_reminder, list_reminders, delete_reminder, setup_hook, on_ready, on_guild_join, on_guild_remove, and on_reaction_add

In scope:
- All Discord adapter modules in monorepo/tlt/adapters/discord_adapter/
- FastAPI router endpoints that trigger Discord messaging operations
- Bot lifecycle hooks (setup_hook, on_ready, on_guild_join, on_guild_remove, on_reaction_add)
- Reminder scheduling and notification delivery workflows
- State synchronization tasks between Discord and TLT service backend

Out of scope:
- Synchronous CLI tools or scripts that do not interact with Discord
- Internal TLT service business logic that does not cross integration boundaries
- Database operations within the TLT service core (unless triggered by Discord events)
- Non-Discord external integrations that may use different async patterns

Exceptions:
- EXC-001: Synchronous wrapper functions are required for testing or CLI tooling that mocks Discord interactions

## Rationale

- Evidence shows consistent use of async/await patterns across 2 adapter modules (reminder.py, bot_manager.py) with 92.20% confidence, indicating an established architectural pattern rather than isolated implementation choices
- Discord's client library enforces async I/O for all messaging operations (channel.send, thread.send, user.send), making async boundaries a technical requirement rather than an optional design choice
- The integration layer coordinates between FastAPI's async request handling and Discord's event-driven bot lifecycle, requiring explicit async patterns to prevent event loop blocking and maintain responsiveness
- Structured validation with Pydantic models (ReminderCreate, ReminderResponse) at integration boundaries provides type safety and input validation before async operations, reducing runtime errors in async workflows

## Consequences

Positive:
- Non-blocking I/O operations enable the Discord bot to handle multiple concurrent events (messages, reactions, reminders) without degrading responsiveness or creating backpressure
- Explicit async boundaries between HTTP and event-driven patterns provide clear architectural separation, making the codebase easier to reason about and test
- Async HTTP clients (aiohttp, httpx) for backend communication prevent blocking the Discord event loop during state synchronization or API calls
- Structured logging and Pydantic validation at async boundaries improve observability and error handling in concurrent workflows

Negative:
- Async/await patterns increase cognitive complexity for developers unfamiliar with Python's asyncio model, requiring additional training and code review attention
- Mixing async and sync code requires careful coordination to avoid blocking operations, deadlocks, or event loop starvation, increasing the risk of subtle concurrency bugs
- Testing async integration boundaries requires async test frameworks (pytest-asyncio) and mock strategies that simulate async I/O, adding complexity to the test suite
- Debugging async workflows across integration boundaries is more challenging due to non-linear execution flow and potential race conditions in state synchronization

## Alternatives

- Use synchronous Discord library with threading for concurrent operations (rejected)
  Rejected because: Discord.py's modern API is async-first, and threading introduces higher overhead, more complex state management, and GIL contention compared to asyncio's cooperative multitasking
  When valid: Only valid for legacy Discord libraries or environments where asyncio is unavailable
- Implement message queue (Redis, RabbitMQ) between FastAPI and Discord bot to decouple async boundaries (deferred)
  Rejected because: Adds infrastructure complexity and latency for a pattern that can be handled with direct async coordination; may be reconsidered if scaling requirements demand horizontal distribution
  When valid: Valid when Discord bot must scale horizontally across multiple processes or when message delivery guarantees require persistent queuing
- Run Discord bot in separate process with IPC communication to isolate async runtime (rejected)
  Rejected because: Process isolation adds IPC overhead and complicates deployment without providing significant benefits given that FastAPI and Discord.py both use asyncio and can share an event loop
  When valid: Valid if Discord bot requires different Python version, dependency isolation, or independent restart cycles

## Risks

- Blocking operations accidentally introduced in async code paths (e.g., synchronous file I/O, blocking HTTP calls) can stall the entire event loop and degrade Discord bot responsiveness
  Mitigation: Enforce code review checks for blocking operations, use async linters (pylint-asyncio), and implement monitoring for event loop lag metrics
  Owner: Engineering team
- State synchronization between Discord events and TLT backend through periodic polling may introduce race conditions or stale data if polling interval is too long or event ordering is not preserved
  Mitigation: Implement idempotent state updates, use versioning or timestamps for conflict resolution, and tune STATE_QUERY_INTERVAL based on observed latency requirements
  Owner: Engineering team
- Unhandled exceptions in async tasks (reminder checks, state polling) may silently fail without crashing the bot, leading to missed reminders or state drift
  Mitigation: Wrap all async tasks with exception handlers, implement dead letter logging for failed operations, and add health check endpoints that verify task liveness
  Owner: Engineering team

## Implementation Notes

- Use asyncio.create_task() for fire-and-forget operations like reminder scheduling, and await directly for operations requiring immediate results like message sending in request handlers
- Configure aiohttp.ClientSession or httpx.AsyncClient with connection pooling and timeout settings to prevent resource exhaustion during backend API calls
- Implement graceful shutdown handlers that await pending tasks and close async resources (HTTP clients, Discord connections) to prevent resource leaks
- Use Pydantic's BaseModel for all data crossing integration boundaries to ensure validation happens before async operations begin, reducing mid-flight errors

## Continuation Context


Verify commands:
- grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l
- grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l
- grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l
- grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l

Accept when:
- All Discord messaging operations (channel.send, thread.send, user.send) are prefixed with 'await' keyword in adapter modules
- All public API contracts in Discord adapter modules use 'async def' function signatures
- External HTTP client usage shows AsyncClient or ClientSession patterns rather than synchronous requests
- Pydantic BaseModel schemas are defined for data structures crossing integration boundaries (ReminderCreate, ReminderResponse)

## Enforcement

- Verified by: Automated code review checks using pylint-asyncio to detect blocking operations in async functions
- Verified by: CI pipeline grep-based verification commands to ensure async patterns are present in adapter modules
- Verified by: Manual code review checklist requiring verification of async/await usage at integration boundaries
- Verified by: Runtime monitoring of event loop lag metrics to detect blocking operations in production
- Violation handling: CI pipeline fails if verification commands do not meet acceptance criteria thresholds
- Violation handling: Code review blocks merge if blocking operations are detected in async code paths without justification
- Violation handling: Runtime alerts trigger if event loop lag exceeds threshold, requiring investigation and remediation
- Violation handling: Quarterly architecture review audits adapter modules for compliance with async boundary patterns
- Exception process: Developer submits exception request with technical justification for synchronous operation requirement
- Exception process: Tech lead reviews exception against policy_exceptions criteria (testing, CLI tooling)
- Exception process: Approved exceptions must document sync-to-async bridge pattern and add inline comments explaining deviation
- Exception process: Exception log maintained in architecture decision log with review date for periodic reassessment