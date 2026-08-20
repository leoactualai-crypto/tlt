# Adopt Python async/await Concurrency Model for Agent Nodes and Discord Handlers: When Calling Async Methods Other Contexts

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent nodes, Discord adapter handlers, and service endpoints.

## Context

- The project integrates with Discord.py, which requires asynchronous handlers for interaction callbacks, event listeners, and message processing
- Agent nodes coordinate multiple I/O-bound operations including Discord API calls, message fetching, rate limiting, and external service communication
- The ambient event agent architecture processes events through a graph of nodes that can execute concurrently when dependencies allow
- Blocking synchronous operations in Discord handlers would prevent the bot from responding to multiple interactions simultaneously
- Python's async/await provides native language support for cooperative multitasking without thread management complexity

## Problem Statement

The system must handle concurrent Discord interactions, agent node execution, and I/O operations without blocking. Synchronous code would serialize all operations, causing unacceptable latency when processing multiple events or waiting for external API responses. The architecture requires a concurrency model that integrates with Discord.py's async event loop while enabling agent nodes to coordinate asynchronously.

## Decision

1. SHOULD: When calling async methods from other async contexts, implementations SHOULD use await rather than creating tasks unless explicit concurrent execution is required

## Policy Block

- SHOULD When calling async methods from other async contexts, implementations SHOULD use await rather than creating tasks unless explicit concurrent execution is required

In scope:
- All classes inheriting from BaseNode in the agent node architecture
- All Discord command handlers, modal submission handlers, button callbacks, and select menu callbacks
- All API route handlers in MCP services
- All message moderation enforcers and event listeners
- Any component performing I/O operations including network requests, database queries, or file system access in the agent or adapter layers

Out of scope:
- Pure computation functions with no I/O that complete in microseconds
- Data model classes and type definitions
- Utility functions for string formatting, data transformation, or validation that do not perform I/O
- Configuration loading at application startup before the event loop starts

## Rationale

- Discord.py's architecture is built on asyncio and requires async handlers for all interaction callbacks and event listeners
- Agent nodes frequently perform I/O-bound operations (Discord API calls, message fetching, rate limit tracking) that benefit from non-blocking execution
- The ambient event agent processes multiple events concurrently, and async/await enables nodes to yield control during I/O waits
- Python's native async/await syntax provides clear, readable concurrency without explicit thread or callback management

## Consequences

Positive:
- Discord bot can handle multiple interactions concurrently without blocking on I/O operations
- Agent nodes can execute in parallel when dependencies allow, improving throughput for event processing
- Rate limiting and delays use non-blocking sleep, allowing other operations to proceed
- Code remains readable with linear async/await syntax rather than callback chains or thread synchronization primitives

Negative:
- All dependencies must be async-compatible or wrapped in executor threads for blocking operations
- Developers must understand async/await semantics and avoid accidentally blocking the event loop
- Debugging async code can be more complex than synchronous code due to interleaved execution
- Testing requires async test frameworks and careful management of event loop lifecycle

## Alternatives

- Use synchronous code with threading for concurrency (rejected)
  Rejected because: Discord.py requires async handlers, and mixing threading with asyncio introduces complexity and race conditions. Thread synchronization overhead would negate performance benefits.
  When valid: For CPU-bound operations that need true parallelism, use executor threads from async context
- Use callback-based asynchronous patterns without async/await (rejected)
  Rejected because: Callback-based code is harder to read and maintain than async/await linear syntax. Python's async/await is the idiomatic modern approach.
  When valid: Never for new code; only when integrating legacy callback-based libraries
- Use multiprocessing for concurrency (rejected)
  Rejected because: Multiprocessing has high overhead for process creation and inter-process communication. Discord.py's event loop cannot span processes.
  When valid: For CPU-intensive tasks that need true parallelism and can be isolated from the main event loop

## Risks

- Accidentally introducing blocking I/O operations that stall the event loop and degrade responsiveness
  Mitigation: Code review checklist includes verification that all I/O uses async libraries. Linting rules detect synchronous I/O in async contexts.
  Owner: engineering team
- Async context propagation errors where sync code calls async methods without await or vice versa
  Mitigation: Type checking with mypy configured to detect async/await mismatches. Abstract base classes enforce async signatures.
  Owner: engineering team
- Unhandled exceptions in async tasks may be silently swallowed if not properly awaited
  Mitigation: All async method calls use await rather than fire-and-forget tasks. Exception handling wraps async operations in try/except blocks.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When implementing new agent nodes, inherit from BaseNode and declare execute as async def. Use await when calling other async methods within the node.
- For Discord interaction handlers, declare callback methods as async def and use await for interaction.response methods and Discord API calls.
- When rate limiting or introducing delays, use the async sleep function from the standard library's async module rather than blocking sleep.
- If integrating a synchronous library that performs blocking I/O, wrap calls in an executor to prevent blocking the event loop.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and run the type checker to verify async method signatures match their declarations
- Discover the project's linting configuration and run the linter with rules that detect blocking I/O in async contexts
- Discover the project's test suite and run async tests to verify agent nodes and handlers execute without blocking

Accept when:
- All agent node execute methods are declared as async def and type checking passes without async/await mismatches
- All Discord interaction handlers (callbacks, modal submissions, button handlers) are declared as async def
- Static analysis confirms no blocking I/O operations are used in async contexts
- Test suite executes async tests successfully and verifies concurrent execution behavior

## Enforcement

- Verified by: Type checking in continuous integration verifies async method signatures
- Verified by: Linting rules detect blocking I/O in async contexts
- Verified by: Code review checklist includes verification of async/await usage
- Verified by: Async test suite validates concurrent execution behavior
- Violation handling: Type checking failures block pull request merge
- Violation handling: Linting violations trigger build warnings and require justification
- Violation handling: Code review identifies blocking operations and requests refactoring
- Violation handling: Runtime monitoring logs event loop blocking warnings in development
- Exception process: Document the blocking operation and why it cannot be made async
- Exception process: Wrap blocking calls in executor threads with explicit justification
- Exception process: Add inline comments explaining the exception and mitigation strategy
- Exception process: Review exception with team lead to confirm no async alternative exists