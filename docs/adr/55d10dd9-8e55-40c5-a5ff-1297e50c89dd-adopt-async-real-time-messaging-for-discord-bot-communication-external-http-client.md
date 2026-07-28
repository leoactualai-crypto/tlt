# Adopt Async Real-Time Messaging for Discord Bot Communication: External Http Client

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all Discord adapter components within the TLT monorepo that interact with real-time messaging boundaries.

## Context

- The TLT Discord adapter requires bidirectional real-time communication with Discord's API for bot operations including message sending, reaction handling, and reminder delivery
- Discord's API is inherently asynchronous and event-driven, requiring async/await patterns for all I/O operations including channel.send(), user.send(), and thread.send()
- The bot_manager.py and reminder.py modules coordinate multiple concurrent operations including periodic state queries, reminder scheduling, and event-driven message handling
- Integration with external TLT services requires non-blocking HTTP client operations using aiohttp and httpx.AsyncClient to prevent blocking the Discord event loop
- The system must maintain responsiveness across guild events (on_ready, on_guild_join, on_reaction_add) while performing background tasks like reminder checks and state monitoring

## Problem Statement

Discord bot adapters must handle multiple concurrent real-time communication streams (user messages, channel updates, reactions, reminders) without blocking, while coordinating with external services and maintaining low-latency responses. Synchronous I/O patterns would block the event loop, causing unacceptable delays in message delivery and event handling.

## Decision

1. MUST: External HTTP client operations to TLT services MUST use async HTTP clients (httpx.AsyncClient, aiohttp) within async context managers

## Policy Block

- MUST External HTTP client operations to TLT services MUST use async HTTP clients (httpx.AsyncClient, aiohttp) within async context managers

In scope:
- All Discord adapter modules (bot_manager.py, reminder.py) within monorepo/tlt/adapters/discord_adapter/
- Discord event handlers and lifecycle hooks (on_ready, on_guild_join, on_reaction_add, setup_hook)
- Real-time messaging operations (channel.send(), user.send(), thread.send())
- Background tasks for reminder scheduling and state monitoring
- FastAPI route handlers that interact with Discord bot instances
- HTTP client operations to external TLT services

Out of scope:
- Pure computational functions with no I/O operations
- Data validation classes (Pydantic BaseModel subclasses like ReminderCreate, ReminderResponse)
- Configuration loading from environment variables (os.getenv) at module initialization
- Logger instantiation (logging.getLogger(__name__))
- Synchronous utility functions that do not interact with Discord API or external services

Exceptions:
- EXC-001: Initialization code that runs before the asyncio event loop starts (module-level configuration loading, logger setup)
- EXC-002: CPU-bound operations that must run in thread pools via asyncio.to_thread() or loop.run_in_executor()

## Rationale

- Evidence shows consistent use of async/await patterns across bot_manager.py and reminder.py with async event handlers (on_ready, on_guild_join, on_reaction_add) and async messaging operations (channel.send(), user.send(), thread.send())
- The detection of httpx.AsyncClient() and aiohttp in boundaries.external_clients demonstrates commitment to non-blocking external service communication, preventing event loop blocking during TLT service queries
- Background task patterns (self.reminder_check_task.start()) and concurrent FastAPI route handlers indicate the system must handle multiple simultaneous operations without mutual blocking
- The paradigm.concurrency_model facet detection across both files with 92.20% confidence substantiates this as an established architectural pattern rather than isolated implementation

## Consequences

Positive:
- Non-blocking I/O enables the bot to handle multiple concurrent Discord events and user interactions without latency degradation
- Async HTTP clients allow parallel queries to TLT services while maintaining Discord event responsiveness
- Background tasks can run continuously (reminder checks, state monitoring) without interfering with real-time message handling
- FastAPI async route handlers enable the REST API to remain responsive under concurrent reminder creation and deletion requests

Negative:
- Async/await syntax increases code complexity and requires developers to understand asyncio event loop semantics and coroutine lifecycle
- Debugging async code is more challenging due to non-linear execution flow and potential for subtle race conditions or deadlocks
- Mixing sync and async code requires careful coordination with thread pools or sync-to-async adapters, increasing cognitive overhead
- Stack traces in async contexts are often less readable and harder to trace through multiple coroutine frames

## Alternatives

- Use synchronous Discord library (discord.py sync fork) with threading for concurrency (rejected)
  Rejected because: Threading model would require complex thread synchronization, increase memory overhead per thread, and discord.py's primary API is async-first with better community support and maintenance
  When valid: Only valid for legacy systems with existing synchronous codebases that cannot be migrated
- Use multiprocessing with separate processes for Discord bot and web API (rejected)
  Rejected because: Inter-process communication overhead would add latency to reminder operations, complicate shared state management (active_reminders), and increase deployment complexity
  When valid: Valid for systems requiring true CPU parallelism or strict isolation between bot and API components
- Hybrid approach with sync FastAPI routes and background async Discord operations (rejected)
  Rejected because: Would require sync-to-async bridges (asyncio.run() or loop.run_until_complete()) that can cause event loop conflicts and deadlocks when FastAPI routes need to interact with bot state
  When valid: Valid only if FastAPI routes never directly interact with Discord bot instances

## Risks

- Unhandled exceptions in async tasks can silently fail without crashing the bot, leading to degraded functionality (e.g., reminders not firing)
  Mitigation: Wrap all async tasks with try-except blocks, log exceptions with full stack traces, and implement health check endpoints that verify background task status
  Owner: Discord Adapter Team
- Blocking operations accidentally introduced in async code (e.g., synchronous database calls, time.sleep()) will block the entire event loop
  Mitigation: Implement code review checklist for async patterns, use static analysis tools to detect blocking calls, and add integration tests that measure event loop responsiveness
  Owner: Engineering Team
- Race conditions in shared state (active_reminders dictionary) accessed from multiple async contexts without proper synchronization
  Mitigation: Use asyncio.Lock for critical sections, document thread-safety requirements, and add concurrency tests that simulate simultaneous reminder operations
  Owner: Discord Adapter Team

## Implementation Notes

- Use discord.py's @tasks.loop() decorator for periodic background operations like reminder checks and state monitoring, ensuring proper error handling and restart logic
- Wrap all httpx.AsyncClient usage in async context managers (async with httpx.AsyncClient() as client:) to ensure proper connection cleanup
- For CPU-bound operations that must run synchronously, use asyncio.to_thread() or loop.run_in_executor() to offload work to thread pools without blocking the event loop
- Implement structured logging with correlation IDs to trace async operations across event handlers, background tasks, and API routes
- Add timeout parameters to all external HTTP requests (httpx.AsyncClient(timeout=...)) to prevent indefinite hangs that would block the event loop

## Continuation Context


Verify commands:
- grep -r 'def on_\|def setup_hook' monorepo/tlt/adapters/discord_adapter/ | grep -v 'async def' && echo 'FAIL: Found non-async event handlers' || echo 'PASS: All event handlers are async'
- grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' && echo 'FAIL: Found non-awaited send operations' || echo 'PASS: All send operations are awaited'
- grep -r 'import requests\|time\.sleep\|open(' monorepo/tlt/adapters/discord_adapter/*.py && echo 'FAIL: Found blocking operations' || echo 'PASS: No blocking operations detected'

Accept when:
- All Discord event handlers (on_ready, on_guild_join, on_reaction_add, setup_hook) are declared as async functions
- All Discord messaging operations (channel.send(), user.send(), thread.send()) are preceded by await keyword
- No synchronous blocking libraries (requests, time.sleep, synchronous file I/O) are imported or used in async contexts
- All external HTTP client operations use async clients (httpx.AsyncClient, aiohttp) within async context managers

## Enforcement

- Verified by: Pre-commit hooks running grep-based pattern detection for blocking operations and non-async event handlers
- Verified by: Code review checklist requiring verification of async/await usage in all Discord API interactions
- Verified by: CI pipeline integration tests measuring event loop responsiveness under concurrent load
- Violation handling: Pre-commit hooks block commits containing synchronous blocking operations in async contexts
- Violation handling: Code review requires mandatory changes before merge if async patterns are violated
- Violation handling: CI failures on integration tests trigger automatic PR comments with remediation guidance
- Exception process: Developer documents exception rationale in code comments and PR description
- Exception process: Tech Lead reviews exception request and validates it matches documented exception criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions are recorded in architecture decision log with expiration date for re-evaluation