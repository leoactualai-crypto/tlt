# Discord.py Interaction Response API for Command and UI Component Messaging: Handlers Use Interaction Response Defer Long

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- Discord's interaction model enforces a 3-second response window using interaction tokens, requiring a specific API pattern for command and UI component responses
- The Discord adapter layer handles slash commands, button interactions, dropdown selections, and modal submissions across multiple command handler modules
- The interaction response API provides message queue semantics with support for ephemeral messaging, deferred responses, and modal workflows
- Direct channel messaging APIs bypass the interaction token contract and fail to provide ephemeral message capabilities required for user-specific feedback

## Problem Statement

Discord bot command handlers and UI components must respond to user interactions within a 3-second window using interaction tokens. Without a standardized message queue abstraction, implementations may use incompatible messaging APIs that violate Discord's interaction contract, fail to support ephemeral messaging, or create inconsistent user experiences across command handlers.

## Decision

1. MAY: Handlers MAY use interaction.response.defer() for long-running operations that cannot complete within the 3-second interaction window, followed by interaction.followup.send

## Policy Block

- MAY Handlers MAY use interaction.response.defer() for long-running operations that cannot complete within the 3-second interaction window, followed by interaction.followup.send

In scope:
- All Discord slash command handlers in the adapter layer
- All Discord UI component callbacks (buttons, dropdowns, select menus, modals)
- Any code path that receives a discord.Interaction object and must respond to the user
- Command routers that dispatch to handler implementations

Out of scope:
- Background tasks or scheduled jobs that send messages without user interaction context
- Event listeners that respond to Discord gateway events (on_message, on_member_join) rather than interactions
- Webhook-based messaging where no interaction token exists
- Follow-up messages sent after the initial interaction response has been completed

## Rationale

- Discord's interaction model requires responses within 3 seconds using the interaction token, and the interaction response API is the only mechanism that satisfies this contract while providing ephemeral messaging capabilities
- Evidence shows consistent adoption across 4 command handler files with significance 0.90-0.91, indicating an established architectural pattern in the Discord adapter layer
- The interaction response API provides a message queue abstraction that handles token-based delivery, deferred responses, and modal workflows, centralizing Discord's interaction contract enforcement
- Ephemeral messaging observed in error handling patterns enables user-specific feedback without channel pollution, a capability unavailable through direct channel messaging APIs

## Consequences

Positive:
- Consistent interaction response pattern across all command handlers ensures compliance with Discord's 3-second interaction token contract
- Ephemeral messaging support enables user-specific error messages and validation feedback without polluting shared channels
- Modal workflow support through interaction.response.send_modal enables multi-step user input flows with maintained interaction context
- Message queue semantics abstract Discord's token-based response delivery, simplifying handler implementation

Negative:
- Interaction response API is tightly coupled to Discord's interaction model, limiting portability to other chat platforms
- 3-second response window requires careful handling of long-running operations through deferred responses, adding complexity
- Interaction tokens are single-use for initial responses, requiring developers to understand the distinction between initial responses and follow-up messages
- Ephemeral messages cannot be edited or deleted by other users, limiting moderation capabilities for user-specific feedback

## Alternatives

- Use direct channel messaging APIs (channel.send) for all command responses (rejected)
  Rejected because: Direct channel messaging bypasses the interaction token contract, fails to provide ephemeral messaging capabilities, and violates Discord's interaction response requirements, causing interaction failures
  When valid: Only valid for background tasks or event listeners that do not originate from user interactions and have no interaction token
- Use webhook-based messaging for command responses (rejected)
  Rejected because: Webhooks do not have access to interaction tokens and cannot provide ephemeral messaging or modal workflows, and require separate webhook creation and management overhead
  When valid: Valid for external integrations or scheduled notifications where no interaction context exists
- Mix interaction responses and direct channel messaging based on message type (rejected)
  Rejected because: Inconsistent messaging patterns across handlers create maintenance burden, violate Discord's interaction contract for some responses, and make it unclear which API to use for new handlers
  When valid: Never valid for initial interaction responses; only valid for follow-up messages after the initial interaction response is complete

## Risks

- Long-running command operations may exceed the 3-second interaction window, causing interaction token expiration and response failures
  Mitigation: Implement deferred response pattern using interaction.response.defer() for operations that cannot complete within 3 seconds, followed by interaction.followup.send with results
  Owner: Discord adapter engineering team
- Developers unfamiliar with Discord's interaction model may attempt to call interaction.response methods multiple times, causing API errors due to single-use token constraint
  Mitigation: Document the distinction between initial responses (interaction.response.*) and follow-up messages (interaction.followup.*) in implementation guides and code review checklists
  Owner: Discord adapter engineering team
- Version changes in the Discord library may alter interaction response API signatures or behavior, breaking existing handler implementations
  Mitigation: Enforce lock-file version verification before implementation and maintain integration tests that validate interaction response patterns against the locked library version
  Owner: Discord adapter engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use interaction.response.edit_message to update the existing message state rather than creating new messages
- Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible
- For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks, as interaction tokens do not persist state between responses

## Continuation Context


Verify commands:
- Discover the project's test suite location and execute integration tests that validate interaction response patterns in command handlers
- Discover the project's static analysis configuration and run type checking to verify all interaction.response API calls match the locked library version's type signatures
- Discover the project's linting configuration and verify that direct channel messaging APIs are flagged when used in command handler or UI callback contexts

Accept when:
- All command handlers and UI component callbacks use interaction.response.* methods for initial responses, with no direct channel.send() calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery

## Enforcement

- Verified by: Code review checklist requiring verification that all new command handlers and UI callbacks use interaction response APIs
- Verified by: Integration tests validating interaction response patterns for each command handler module
- Verified by: Static analysis rules flagging direct channel messaging in interaction callback contexts
- Violation handling: Code review rejection for pull requests that use direct channel messaging in command handlers or UI callbacks
- Violation handling: CI pipeline failure when integration tests detect interaction response pattern violations
- Violation handling: Runtime warnings logged when interaction tokens expire due to missing deferred response patterns
- Exception process: Document the specific use case requiring an exception (background task, event listener, webhook integration)
- Exception process: Obtain approval from Discord adapter team lead confirming no interaction token exists in the context
- Exception process: Add inline comment explaining why direct channel messaging is required and referencing this ADR's policy scope exclusions