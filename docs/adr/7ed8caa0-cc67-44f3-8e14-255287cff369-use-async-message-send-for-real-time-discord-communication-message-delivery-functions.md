# Use Async Message Send for Real-Time Discord Communication: Message Delivery Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all Discord adapter components that send messages to channels, threads, or users.

## Context

- The Discord adapter components (reminder.py and bot_manager.py) require non-blocking communication with Discord's API to maintain responsiveness during message delivery operations
- The system handles multiple concurrent operations including reminder scheduling, state monitoring, and event updates that must not block each other
- Discord's API client library (discord.py) provides async/await primitives for all I/O operations, establishing an async-first programming model
- Real-time user interactions (reminders, reactions, guild events) demand immediate feedback without blocking the event loop or delaying other operations

## Problem Statement

Discord adapter components must send messages to channels, threads, and users without blocking the event loop or degrading responsiveness of concurrent operations such as reminder scheduling, state monitoring, and reaction handling. Synchronous I/O would introduce latency cascades and reduce system throughput.

## Decision

1. MUST: Message delivery functions MUST be declared as async def to support non-blocking I/O

## Policy Block

- MUST Message delivery functions MUST be declared as async def to support non-blocking I/O

In scope:
- All Discord message send operations in reminder.py and bot_manager.py
- HTTP client operations to TLT service endpoints
- Discord event handlers (on_ready, on_guild_join, on_reaction_add)
- Reminder scheduling and delivery functions
- State monitoring and event update polling

Out of scope:
- Synchronous utility functions that do not perform I/O
- Data model definitions (Pydantic BaseModel classes)
- Configuration loading from environment variables
- In-memory data structure operations (dictionary lookups, list operations)

Exceptions:
- EXC-001: Initialization code that runs before the event loop starts

## Rationale

- Evidence shows consistent use of await with send operations (await thread.send(content), await channel.send(content), await user.send(message)) across both reminder.py and bot_manager.py
- The discord.py library enforces async patterns for all I/O operations, making async/await the idiomatic approach for Discord bot development
- Concurrent operations (reminder checks, state monitoring, event handling) require non-blocking I/O to maintain system responsiveness and throughput
- The pattern appears in 2 files with 92.20% confidence, indicating consistent architectural practice within the Discord adapter boundary

## Consequences

Positive:
- Non-blocking message delivery maintains event loop responsiveness during high-volume operations
- Multiple concurrent operations (reminders, state queries, event handling) can execute in parallel without blocking each other
- Aligns with discord.py library idioms, reducing impedance mismatch and simplifying integration
- Enables efficient resource utilization through cooperative multitasking rather than thread-based concurrency

Negative:
- Async/await syntax increases code complexity for developers unfamiliar with asynchronous programming
- Debugging async code requires understanding of event loop mechanics and coroutine lifecycle
- Mixing sync and async code requires careful coordination and can introduce subtle bugs
- Error handling in async contexts requires try/except within async functions, increasing boilerplate

## Alternatives

- Use synchronous blocking calls with threading for concurrency (rejected)
  Rejected because: Threading introduces higher memory overhead, GIL contention, and does not align with discord.py's async-first API design. Would require wrapping all Discord API calls in thread executors.
  When valid: Only valid for legacy integrations where async refactoring is prohibitively expensive
- Use callback-based asynchronous patterns instead of async/await (rejected)
  Rejected because: Callback-based patterns lead to callback hell, reduce code readability, and are not supported by discord.py's modern API surface
  When valid: Not applicable for Python 3.7+ codebases with native async/await support
- Hybrid approach with sync wrappers around async operations (rejected)
  Rejected because: Sync wrappers (asyncio.run) block the event loop and defeat the purpose of async I/O, causing performance degradation
  When valid: Only for isolated CLI tools or scripts that do not run within an existing event loop

## Risks

- Developers unfamiliar with async/await may introduce blocking calls that degrade performance
  Mitigation: Provide async/await training, code review checklist for blocking operations, and linting rules to detect synchronous I/O in async contexts
  Owner: Engineering team
- Unhandled exceptions in async tasks may silently fail without proper error handling
  Mitigation: Implement centralized error handling for async tasks, use asyncio exception handlers, and ensure all tasks have try/except blocks with logging
  Owner: Engineering team
- Event loop blocking due to CPU-intensive operations in async functions
  Mitigation: Profile async functions to identify CPU-bound operations and move them to thread/process executors using asyncio.run_in_executor
  Owner: Engineering team

## Implementation Notes

- All new Discord message send operations must use await with channel.send(), thread.send(), or user.send() methods
- HTTP client operations should use aiohttp.AsyncClient with async context managers (async with) for proper resource cleanup
- Use asyncio.gather() to parallelize independent async operations such as sending multiple messages or querying multiple endpoints
- Implement proper exception handling within async functions using try/except blocks and log errors using the logging module (logging.getLogger(__name__))

## Continuation Context


Verify commands:
- grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l
- grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l
- python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v

Accept when:
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings

## Enforcement

- Verified by: Code review checklist requiring verification of async/await usage for all I/O operations
- Verified by: Static analysis using pylint async rules (e.g., not-async-context-manager, await-outside-async)
- Verified by: Integration tests that measure event loop responsiveness and detect blocking operations
- Violation handling: CI pipeline fails if synchronous Discord API calls are detected in async contexts
- Violation handling: Code review blocks merge if blocking I/O is found in async functions without justification
- Violation handling: Performance regression tests flag PRs that introduce event loop blocking
- Exception process: Document the specific reason why async/await cannot be used (e.g., initialization code, pure computation)
- Exception process: Obtain tech lead approval with confirmation that no I/O occurs in the synchronous code path
- Exception process: Add inline comment explaining the exception and link to approval in PR discussion