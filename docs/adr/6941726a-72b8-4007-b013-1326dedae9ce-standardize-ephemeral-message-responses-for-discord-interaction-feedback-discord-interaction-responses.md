# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Discord Interaction Responses

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Discord adapter command handlers consistently use ephemeral message responses for user feedback across create, update, delete, list, info, and vibe operations
- The pattern appears in 6 handler files within the monorepo/tlt/adapters/discord_adapter/commands/tlt/ directory, indicating systematic adoption
- Message queue boundaries are established through interaction.response.send_message() calls with ephemeral=True parameter for error states, empty states, and success confirmations
- The codebase uses discord library for bot interactions and requires consistent user experience across all command handlers
- Integration testing relies on event.get('guild_id') pattern and message response behavior for validation

## Problem Statement

Discord bot command handlers require consistent feedback mechanisms that do not clutter shared channels while providing clear user guidance. Without standardized ephemeral messaging patterns, error states, empty results, and success confirmations may create channel noise, inconsistent user experience, and complicate integration testing of message queue boundaries.

## Decision

1. MUST: All Discord interaction responses for error states MUST use ephemeral=True parameter to prevent channel clutter

## Policy Block

- MUST All Discord interaction responses for error states MUST use ephemeral=True parameter to prevent channel clutter

In scope:
- Discord adapter command handlers in monorepo/tlt/adapters/discord_adapter/commands/
- All interaction.response.send_message() and related response methods
- Error handling, empty state handling, and success confirmation flows
- Modal submission handlers and view interaction callbacks

Out of scope:
- Non-Discord adapters or alternative messaging platforms
- System-to-system logging or observability messages
- Webhook-based message delivery outside interaction context
- Direct message (DM) flows where ephemeral parameter is not applicable

Exceptions:
- EXC-001: Public event announcements or channel-wide notifications are explicitly required by the feature specification

## Rationale

- The evidence shows 6 handler files consistently applying ephemeral=True across error states, empty results, and user feedback, indicating an established pattern with 91.52% confidence
- Ephemeral messaging prevents channel clutter and provides private feedback, improving user experience in shared Discord environments
- Standardizing message queue boundary behavior (boundaries.message_queues facet) enables predictable integration testing through event.get('guild_id') and response validation patterns
- The discord library's interaction model supports ephemeral responses natively, making this pattern a natural fit for the platform's capabilities

## Consequences

Positive:
- Reduced channel clutter as error messages, empty states, and confirmations remain private to command initiators
- Consistent user experience across all command handlers (create, update, delete, list, info, vibe)
- Simplified integration testing with predictable message queue boundary behavior
- Improved privacy for user actions and command feedback in shared server environments

Negative:
- Ephemeral messages cannot be referenced or quoted by other users, limiting collaborative debugging scenarios
- Users cannot easily share error messages or feedback with moderators for support purposes
- Ephemeral messages disappear on client restart, potentially losing important feedback if not immediately read
- Testing ephemeral message delivery requires mocking interaction response objects, adding test complexity

## Alternatives

- Use non-ephemeral messages for all responses to maintain message history (rejected)
  Rejected because: Creates excessive channel clutter with error messages and user-specific feedback, degrading shared channel experience and violating privacy expectations for command interactions
  When valid: Only valid for explicit public announcements or channel-wide event notifications
- Send all feedback via direct messages (DMs) instead of ephemeral channel responses (rejected)
  Rejected because: Requires additional permissions, may fail if users block DMs, and separates feedback from command context, reducing usability
  When valid: Valid for sensitive information that should not appear in any channel context, even ephemerally
- Mix ephemeral and non-ephemeral responses based on message severity or type (deferred)
  Rejected because: null
  When valid: Could be adopted for specific use cases where public visibility is required (e.g., event creation announcements) while maintaining ephemeral pattern for errors and confirmations

## Risks

- Developers may forget to set ephemeral=True in new command handlers, creating inconsistent user experience
  Mitigation: Implement linting rules or code review checklist items to verify ephemeral parameter usage in all interaction responses
  Owner: engineering team
- Integration tests may not adequately validate ephemeral message behavior, allowing regressions
  Mitigation: Create test utilities that assert ephemeral parameter is set correctly and include in standard test suite for all handlers
  Owner: engineering team
- Users may miss important ephemeral feedback if they navigate away before reading, leading to confusion
  Mitigation: Ensure ephemeral messages are clear, actionable, and include guidance for next steps; consider logging user-facing errors for support purposes
  Owner: product and engineering teams

## Implementation Notes

- Create a base handler class or utility function that wraps interaction.response.send_message() with ephemeral=True as the default parameter
- Document the ephemeral messaging pattern in the Discord adapter README and developer onboarding materials
- Add code review checklist item: 'Verify all interaction responses use ephemeral=True unless explicitly justified'
- Create test fixtures that validate ephemeral parameter is set correctly in mock interaction responses

## Continuation Context


Verify commands:
- grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -v 'ephemeral=True' | grep -v 'ephemeral=False' # Should return only justified non-ephemeral cases
- grep -r 'send_message.*ephemeral=True' monorepo/tlt/adapters/discord_adapter/commands/ | wc -l # Should show consistent usage across handlers
- python -m pytest monorepo/tlt/adapters/discord_adapter/commands/ -k 'test_ephemeral' -v # Run ephemeral message tests

Accept when:
- All interaction.response.send_message() calls in command handlers include explicit ephemeral parameter
- Integration tests validate ephemeral=True for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge

## Enforcement

- Verified by: Code review process with explicit checklist item for ephemeral parameter verification
- Verified by: Automated grep-based verification in CI pipeline checking for interaction.response.send_message patterns
- Verified by: Integration test suite validating ephemeral parameter in mock interaction responses
- Violation handling: CI pipeline fails if interaction.response.send_message() calls lack explicit ephemeral parameter
- Violation handling: Code review blocks merge if ephemeral messaging pattern is violated without documented justification
- Violation handling: Post-merge violations trigger technical debt ticket for remediation
- Exception process: Developer documents rationale for non-ephemeral message in handler docstring and pull request description
- Exception process: Product owner confirms public messaging is required by feature specification
- Exception process: Engineering lead approves exception and adds to policy_exceptions documentation