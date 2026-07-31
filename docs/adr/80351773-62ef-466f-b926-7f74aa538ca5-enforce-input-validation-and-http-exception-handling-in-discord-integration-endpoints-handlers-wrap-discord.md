# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Handlers Wrap Discord

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The Discord adapter exposes HTTP endpoints for handling reaction updates and querying event reactions, requiring validation of message IDs, user IDs, and emoji parameters against bot state
- Integration endpoints coordinate between FastAPI router handlers and Discord bot client operations, necessitating defensive validation at HTTP boundaries before accessing guild, channel, and message resources
- The codebase uses Pydantic BaseModel (ReactionUpdate) for request validation and HTTPException for standardized error responses across 404 (not found) and 500 (server error) cases
- Asynchronous Discord API calls (fetch_message, get_guild, get_channel, get_member) introduce failure modes requiring exception handling and state consistency checks

## Problem Statement

HTTP endpoints that bridge external API requests to Discord bot operations must validate input parameters, verify resource existence, and handle asynchronous failures without exposing internal state or allowing invalid operations to corrupt event tracking data structures.

## Decision

1. MUST: Handlers MUST wrap Discord API operations in try-except blocks and raise HTTPException with status_code=500 and error details on exception

## Policy Block

- MUST Handlers MUST wrap Discord API operations in try-except blocks and raise HTTPException with status_code=500 and error details on exception

## Rationale

- The evidence shows consistent validation patterns across three router endpoints (handle_reaction POST, get_reactions GET, get_reaction_users GET) that check message_id in bot.active_events before proceeding
- Pydantic BaseModel validation (ReactionUpdate class) provides type safety and automatic request parsing, preventing malformed data from reaching business logic
- HTTPException usage with specific status codes (404, 500) creates a standardized error contract for API consumers and prevents information leakage through unhandled exceptions
- Defensive initialization of nested dictionaries (event['reactions'], event['reactions'][emoji]) prevents KeyError exceptions during concurrent reaction updates

## Consequences

Positive:
- Input validation at HTTP boundaries prevents invalid parameters from triggering Discord API calls or corrupting bot state
- Standardized HTTPException responses provide clear error semantics to API consumers and enable proper error handling in client code
- Defensive state initialization prevents race conditions and KeyError exceptions when multiple users react simultaneously
- Structured logging of validation failures and exceptions enables debugging and security monitoring of integration endpoints

Negative:
- Multiple validation checks (event existence, guild, channel, message, user) add latency to each request and increase code verbosity
- Exception handling with HTTPException wrapping obscures original Discord API error details in 500 responses
- Defensive dictionary initialization adds boilerplate to state mutation logic and may mask design issues with state management

## Alternatives

- Use FastAPI dependency injection for validation logic instead of inline checks in each handler (rejected)
  Rejected because: Evidence shows inline validation pattern is already established across all three endpoints; refactoring would require changing existing working code without clear benefit
  When valid: When adding new endpoints or refactoring the router module for maintainability
- Return None or empty responses instead of raising HTTPException for missing resources (rejected)
  Rejected because: HTTP 404 status codes provide standard semantics for resource-not-found conditions; silent failures would complicate client error handling
  When valid: Never for REST API endpoints; only acceptable for internal service-to-service calls with different error handling conventions
- Pre-initialize all event state structures (reactions dictionary) at event creation time (deferred)
  When valid: When refactoring event creation logic; would eliminate defensive initialization in handlers but requires coordinated changes across event lifecycle

## Risks

- Validation logic duplication across endpoints creates maintenance burden and risk of inconsistent validation behavior
  Mitigation: Extract common validation patterns into shared helper functions or FastAPI dependencies; add integration tests covering validation paths
  Owner: engineering team
- HTTPException with status_code=500 and str(e) detail may leak sensitive internal error information to API consumers
  Mitigation: Implement error sanitization that logs full exception details but returns generic error messages in 500 responses; add security review for exception handling
  Owner: security team
- Asynchronous Discord API calls (fetch_message, get_member) may timeout or fail intermittently, causing false 500 errors
  Mitigation: Add retry logic with exponential backoff for transient Discord API failures; implement circuit breaker pattern for degraded Discord API availability
  Owner: engineering team

## Implementation Notes

- Use the ReactionUpdate Pydantic model pattern for all new request bodies requiring validation: define BaseModel subclass with typed fields
- Follow the validation sequence: check bot.active_events first, then validate Discord resources (guild, channel, message, user) in dependency order
- Initialize nested state structures before mutation: if 'reactions' not in event: event['reactions'] = {}; if emoji not in event['reactions']: event['reactions'][emoji] = []
- Log validation failures at info level for user-triggered conditions (user not found, event not found) and error level for system exceptions

## Continuation Context


Verify commands:
- grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'
- grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l
- grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py

Accept when:
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations

## Enforcement

- Verified by: Code review checklist requiring validation checks for all new HTTP endpoints accessing bot state or Discord API
- Verified by: Integration tests covering validation failure paths (missing events, missing Discord resources, malformed requests)
- Verified by: Static analysis rules detecting HTTPException usage patterns and Pydantic model definitions
- Violation handling: Pull requests adding HTTP endpoints without input validation or exception handling are blocked in code review
- Violation handling: Integration test failures for validation paths block merge to main branch
- Violation handling: Production incidents caused by unhandled exceptions trigger post-mortem review and validation pattern enforcement
- Exception process: Exceptions to validation requirements require security team approval and documented risk acceptance
- Exception process: Internal-only endpoints may use simplified validation with approval from tech lead
- Exception process: Exception requests must include alternative mitigation strategy and monitoring plan