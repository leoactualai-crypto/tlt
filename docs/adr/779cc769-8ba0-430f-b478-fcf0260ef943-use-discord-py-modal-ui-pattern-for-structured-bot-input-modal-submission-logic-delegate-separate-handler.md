# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Submission Logic Delegate Separate Handler

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The Discord adapter layer requires structured multi-field user input for event creation and update operations, where users must provide topic, location, and time information
- Discord's native modal UI components provide form-based input within the Discord interface, avoiding the complexity of parsing free-text command arguments or managing multi-step conversation flows
- The codebase implements Discord bot commands using the discord.py library, which exposes modal functionality through the discord.ui.Modal and discord.ui.TextInput APIs
- The pattern appears in both command modal definitions and event bridge handlers, indicating established usage across the Discord adapter boundary

## Problem Statement

Discord bot commands requiring multiple structured input fields need a user-friendly input mechanism that validates field requirements, provides clear prompts, and integrates natively with Discord's interface. Alternative approaches such as parsing command arguments or managing multi-message conversation flows introduce complexity in validation, error handling, and user experience.

## Decision

1. SHOULD: Modal submission logic SHOULD delegate to separate handler classes rather than implementing business logic directly in the on_submit method

## Policy Block

- SHOULD Modal submission logic SHOULD delegate to separate handler classes rather than implementing business logic directly in the on_submit method

In scope:
- Discord bot command implementations within the adapter layer requiring two or more structured input fields
- Event creation, update, or configuration flows where users must provide multiple pieces of information
- Scenarios where input validation and field constraints are required at the UI level

Out of scope:
- Simple commands with single-value input where slash command options suffice
- Read-only commands or queries that do not collect user input
- Non-Discord interaction surfaces such as web interfaces or REST APIs
- Commands where input is provided via message components (buttons, select menus) rather than text fields

## Rationale

- The evidence shows consistent usage of discord.ui.Modal and discord.ui.TextInput across 2 files with significance 0.90, indicating an established architectural pattern rather than experimental usage
- Discord's native modal UI provides built-in validation, field constraints, and user experience consistency within the Discord platform, reducing custom validation logic
- The pattern separates UI definition (modal structure) from business logic (handler delegation), supporting maintainability and testability
- Modal-based input avoids the complexity of parsing free-text command arguments or managing stateful multi-message conversation flows

## Consequences

Positive:
- Native Discord UI integration provides familiar, consistent user experience for Discord users
- Field-level validation and constraints (required, max_length) are enforced at the UI layer before submission
- Structured form definition as class attributes provides clear, declarative input schema
- Separation of modal definition from submission handling supports unit testing and handler reuse

Negative:
- Tight coupling to discord.py's modal API limits portability to other chat platforms or interfaces
- Modal UI is constrained by Discord's limitations (field count, character limits, layout options)
- Async callback pattern requires careful error handling and interaction response management within Discord's timeout constraints
- Changes to discord.py's modal API in major versions may require refactoring across all modal implementations

## Alternatives

- Parse multi-field input from slash command options with individual parameters (rejected)
  Rejected because: Slash command options become unwieldy with many fields, lack rich validation UI, and require users to provide all input upfront without guided prompts
  When valid: Valid for commands with 1-3 simple parameters where inline completion is preferred over form submission
- Implement multi-step conversation flow using message listeners and state management (rejected)
  Rejected because: Conversation flows require complex state management, timeout handling, and provide poor user experience compared to single-form submission
  When valid: Valid for complex wizards requiring conditional branching or dynamic field generation based on previous answers
- Use Discord message components (buttons, select menus) for input collection (rejected)
  Rejected because: Message components are designed for selection and actions, not free-text input, and would require hybrid approaches for text fields
  When valid: Valid for input scenarios involving selection from predefined options rather than free-text entry

## Risks

- Discord API rate limits or interaction timeout constraints may cause modal submissions to fail if handler processing exceeds timeout windows
  Mitigation: Implement immediate interaction acknowledgment with deferred response pattern, offload long-running processing to background tasks
  Owner: engineering team
- Breaking changes in discord.py major versions may alter modal API surface or behavior, requiring coordinated updates across all modal implementations
  Mitigation: Pin discord.py to specific major version range in dependency manifest, establish testing coverage for modal submission flows, document version-specific API usage
  Owner: engineering team
- Modal UI limitations (field count, character limits) may constrain future feature requirements that need more complex input
  Mitigation: Document modal constraints in design phase, consider hybrid approaches (modal for initial input, follow-up interactions for extended data) when requirements exceed modal capabilities
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
- Modal classes should be instantiated with any required bot instance or context dependencies passed to __init__, as modals are created before user interaction occurs
- TextInput field definitions support additional configuration options including style (short/paragraph), default values, and min_length constraints — consult version-specific documentation for available parameters
- Handler delegation pattern should pass interaction object and field values to handler methods, allowing handlers to manage interaction responses (defer, respond, followup) according to processing requirements

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock artifact, resolve the installed discord library version, and verify it matches the environment
- Locate and execute the project's test suite covering modal submission flows to verify on_submit callbacks handle interactions correctly
- Inspect modal class definitions to confirm inheritance from discord.ui.Modal, field definitions using discord.ui.TextInput, and async on_submit implementation

Accept when:
- All modal classes inherit from discord.ui.Modal with title parameter and define fields as discord.ui.TextInput class attributes
- Modal on_submit methods are async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints

## Enforcement

- Verified by: Code review verification that new Discord command implementations requiring multi-field input use discord.ui.Modal pattern
- Verified by: Static analysis or linting rules detecting direct text parsing in command handlers where modal pattern should apply
- Verified by: Test coverage requirements for modal submission flows including interaction handling and handler delegation
- Violation handling: Code review feedback requiring refactoring to modal pattern for multi-field input scenarios within policy scope
- Violation handling: Architecture review escalation for cases where modal limitations prevent compliance, triggering alternative evaluation
- Violation handling: Documentation of exceptions in policy scope when alternative patterns are justified by specific constraints
- Exception process: Developer documents specific constraint preventing modal usage (e.g., field count exceeds Discord limits, dynamic field generation required)
- Exception process: Architecture review evaluates whether constraint is genuine limitation or design issue, proposes alternative approach
- Exception process: Exception approval requires documentation of alternative pattern, rationale, and scope limitation to specific use case